---
title: "Layer 1 – Schedule Guard"
date: 2026-11-18
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Layer 1 – Schedule Guard

Layer 1 stops the lab instance at a fixed time every day. It does not read any cost or billing data, so it is the most reliable of the three layers.

![Layer 1 – Schedule Guard](../../images/5-Workshop/layer1-schedule.png)

### How it works

1. **EventBridge Scheduler** invokes the Lambda function `CostGuard-ScheduleHandler` every day at 22:00 (time zone `Asia/Ho_Chi_Minh`), with the event `{"action": "stop"}`.
2. The function reads the instance ID from `/costguard/instance-id`.
3. It calls `DescribeInstances`. If the instance is `running`, it calls `StopInstances`. In any other state, it does nothing, so a repeated call is safe.
4. It writes a record to `CostGuardSessions` (`schedule_stop` or `schedule_noop`).
5. If anything fails, it publishes the error to `costguard-errors` and raises the error, so the CloudWatch alarm in 5.6 can see it.

The schedule only **stops** the machine. Starting it stays a manual action, so no credit is spent while nobody uses the lab.

### Why EventBridge Scheduler

An EventBridge **rule** reads a cron expression in UTC, so a stop planned for 22:00 in Vietnam must be written as 15:00. EventBridge Scheduler accepts a time zone, so the expression below says exactly what it means. Scheduler also invokes the function through its own role, so the function needs no extra resource-based permission.

```yaml
StopSchedule:
  Type: AWS::Scheduler::Schedule
  Properties:
    Name: costguard-daily-stop
    ScheduleExpression: cron(0 22 * * ? *)
    ScheduleExpressionTimezone: Asia/Ho_Chi_Minh
    FlexibleTimeWindow:
      Mode: "OFF"
    Target:
      Arn: !GetAtt ScheduleHandlerFunction.Arn
      RoleArn: !GetAtt SchedulerRole.Arn
      Input: '{"action": "stop"}'
```

### Step 1 – Look at the schedule

1. Open **EventBridge → Scheduler → Schedules** and choose `costguard-daily-stop`.
2. Check the cron expression, the time zone and the target `CostGuard-ScheduleHandler`.

### Step 2 – Test the function

1. Make sure the instance is **running** (EC2 console).
2. Open **Lambda → CostGuard-ScheduleHandler → Test**. Create a test event with the body `{}` and choose **Test**.
3. Open the **EC2** console and refresh.
4. Open **DynamoDB → Tables → CostGuardSessions → Explore table items**.

**Expected result:** the function returns `schedule_stop` with `state_before` set to `running`. The instance goes from `stopping` to `stopped`. The table has a new item with `event = schedule_stop`.

Run the test a second time while the instance is stopped. **Expected result:** the function returns `schedule_noop` and the instance is not touched.

### Step 3 – Test the real schedule (optional)

1. Start the instance again.
2. Open **CloudFormation → CostGuard → Update**, keep the current template and change `StopSchedule` to a time a few minutes ahead, for example `cron(25 15 * * ? *)` at 15:20 Vietnam time.
3. Wait until that time and check the instance state.
4. Update the stack again and put `cron(0 22 * * ? *)` back.

**Expected result:** the instance is stopped at the chosen time in Vietnam time, not seven hours earlier or later.

### Things to check if something goes wrong

- **The instance does not stop:** check the **Monitor** tab of the function for invocations, and the log group `/aws/lambda/CostGuard-ScheduleHandler` for the error message.
- **The stop runs at the wrong hour:** read the time zone field of the schedule. A schedule without a time zone is read in UTC.
- **`AccessDenied` in the log:** the role of the function can stop only the instance ID that was given to the stack. If you replaced the instance, update the stack with the new `ExistingInstanceId`.
