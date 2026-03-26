---
layout: post
title: locust系列-延时定义user.tasks
key: 202603251504
tags: locust
---

# 版本：locust==2.32.1

locust 要求压测逻辑定义在 `User.tasks` 类变量中。对于基于 TCP 长连接的游戏项目来说，这种方式并不适用。因为只有在角色登录成功并返回角色数据后，压测逻辑才能根据业务数据状态发起请求。

## locust 如时使用 user.tasks

在 locust==2.32.1 源码中，首次使用 task 的流程如下：

```
Runner.spawn_users() - 第215行调用 new_user.start(self.user_greenlets)
User.start() - 第188行通过 group.spawn() 启动 greenlet
run_user() - 内部函数，第186行调用 user.run()
User.run() - 第144行以 DefaultTaskSet 管理 user.tasks
TaskSet.run() - 第335行调用 self.execute_next_task()，读取 user.tasks，并随机执行一个 task
```

按照 locust 的使用规范，`user`是我们能容易访问到的实例 `user.run`，源码如下：

```python
    @final
    def run(self):
        self._state = LOCUST_STATE_RUNNING
        self._taskset_instance = DefaultTaskSet(self)
        try:
            # run the TaskSet on_start method, if it has one
            try:
                self.on_start()
            except Exception as e:
                # unhandled exceptions inside tasks are logged in TaskSet.run, but since we're not yet there...
                logger.error("%s\n%s", e, traceback.format_exc())
                raise

            self._taskset_instance.run()
        except (GreenletExit, StopUser):
            # run the on_stop method, if it has one
            self.on_stop()
```

遗憾的是 `user.run()` 是一个 `final` 方法，无法被重写。因此需要从上层 `user.start` 入手。

```python
  def start(self, group: Group):
        """
        Start a greenlet that runs this User instance.

        :param group: Group instance where the greenlet will be spawned.
        :type group: gevent.pool.Group
        :returns: The spawned greenlet.
        """

        def run_user(user):
            """
            Main function for User greenlet. It's important that this function takes the user
            instance as an argument, since we use greenlet_instance.args[0] to retrieve a reference to the
            User instance.
            """
            user.run()

        self._greenlet = group.spawn(run_user, self)
        self._group = group
        return self._greenlet
```

理论上，可以在自定义的 `MyUser` 中重写 `start` 方法，在调用 `user.run` 之前先检查是否登录成功：

```python
class MyUser(User):

    def is_login_suc():
        ok = False
        # do something here 
        return ok

    def start(self, group: Group):

        def run_user(user):
            # 检查是否登录成功
            if not self.is_login_suc():
                return

            # 登录成功后，才启动压测逻辑
            user.run()

        self._greenlet = group.spawn(run_user, self)
        self._group = group

        return self._greenlet
```

当面向一种更为复杂的情况是：期望部分角色在战斗、部分在养成，或按一定比例分配各类 task 的数量。此时，由类变量预先定义 task 的方式无法满足需求，需要运行时依据策略动态赋值，将类变量转为实例变量。

- `MyUser.tasks` 不能为 `None`，否则启动时 locust 会抛出异常。下例中的 `DefaultTask` 起占位作用：

```python
    def get_next_task(self):
        if not self.tasks:
            if getattr(self, "task", None):
                extra_message = ", but you have set a 'task' attribute - maybe you meant to set 'tasks'?"
            else:
                extra_message = "."
            raise Exception(
                f"No tasks defined on {self.__class__.__name__}{extra_message} use the @task decorator or set the 'tasks' attribute of the TaskSet"
            )
        return random.choice(self.tasks)
```

- `DefaultTask` 必须调用 `self.interrupt(False)`。这是我踩坑后查阅源码才发现的。`user.run` 会创建协程，通过 `while` 循环 + `sleep` 的方式运行所有父子 task。其工作机制是：父 task 停止，同一时刻只有一个子 task 在工作。只有子 task 交出循环控制权，父 task 才能继续执行。如果子 task 一直不交出，就会持续运行该子 task 的内容。`self.interrupt(False)` 的作用正是交出循环控制权，以便登录成功后切换到新的 `self.tasks`。

```python
@final
    def run(self):
        try:
            self.on_start()
        except InterruptTaskSet as e:
            if e.reschedule:
                raise RescheduleTaskImmediately(e.reschedule).with_traceback(e.__traceback__)
            else:
                raise RescheduleTask(e.reschedule).with_traceback(e.__traceback__)

        while True:
            try:
                if not self._task_queue:
                    self.schedule_task(self.get_next_task())

                try:
                    if self.user._state == LOCUST_STATE_STOPPING:
                        raise StopUser()
                    self.execute_next_task()
                except RescheduleTaskImmediately:
                    pass
                except RescheduleTask:
                    self.wait()
                else:
                    self.wait()
            except InterruptTaskSet as e:
                try:
                    self.on_stop()
                except (StopUser, GreenletExit):
                    raise
                except Exception:
                    logging.error("Uncaught exception in on_stop: \n%s", traceback.format_exc())
                if e.reschedule:
                    raise RescheduleTaskImmediately(e.reschedule) from e
                else:
                    raise RescheduleTask(e.reschedule) from e
            except (StopUser, GreenletExit):
                try:
                    self.on_stop()
                except Exception:
                    logging.error("Uncaught exception in on_stop: \n%s", traceback.format_exc())
                raise
            except Exception as e:
                self.user.environment.events.user_error.fire(user_instance=self, exception=e, tb=e.__traceback__)
                if self.user.environment.catch_exceptions:
                    logger.error("%s\n%s", e, traceback.format_exc())
                    self.wait()
                else:
                    raise
```

# 示例代码

```python
from locust import *
import threading

class DefaultTask(TaskSet):
    @task
    def test(self):
        print("default task")
        self.interrupt(False)

class MyTask(TaskSet):

    @task
    def test(self):
        print("my task")

class MyUser(User):
    wait_time = between(1, 2)
    tasks = [DefaultTask]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        def _delay_cb():
            self.tasks = [MyTask]

        threading.Timer(5, _delay_cb).start()
```

<b>原文:<br>
<https://lizijie.github.io/2026/03/14/skynet%E5%AE%9A%E5%88%B6%E5%8A%A0%E8%BD%BDloader.html>
<br>

作者github:<br>
<https://github.com/lizijie>
</b>
