Introduction

The goal of this project is to implement a deadline driven scheduler (DDS) using FreeRTOS. The deadline driven scheduler uses earliest deadline first scheduling (EDF) to dynamically change task priorities based on which has the soonest absolute deadline. A task with the soonest deadline will be given a priority of ‘high’ and all other tasks are given priorities of ‘low’. These tasks are referred to within this project as deadline-driven tasks (DD-tasks).

FreeRTOS does not support deadline driven scheduling; therefore, the algorithm must be built on top of the existing FreeRTOS scheduler. Four FreeRTOS tasks (F-tasks) are used to implement and test the DDS: deadline-driven scheduler task, user-defined tasks, deadline-driven task generator, and the monitor task. Here is a list of F-tasks in order of priority:

The scheduler task has the highest priority and reads from a queue of messages sent from the other three F-tasks. It is responsible for shifting the priorities of the DD-tasks based on their deadlines, creating new DD-tasks, and removing them from the active list once they finish or become overdue. This has the highest task priority.
The monitor task periodically reports information on active tasks, completed tasks, and overdue tasks. This has the second highest task priority.
The task generator periodically creates new DD-tasks using a software timer. The timer will interrupt the current task.
User-defined tasks act as a shell for DD-tasks, allowing them to be actively run. These have the lowest priority.

Design Solution 

The scheduler makes use of a message queue read by the deadline-driven scheduler task (F-task). Different message types are sent to the queue by the other F-tasks and invoke processes within the scheduler. Our design process involved separating the functionality into their corresponding message types and implementing each scheduler functionality one by one. 

Design Document

This is the initial design document that we created. It is a high level illustration of how data moves through the scheduler and the dependencies between scheduler functionality.

Overall, much of the design document is the same as the final design. The only difference is how we handled creating tasks. We didn’t use an F-task for the functionality of creating a task, instead we used a software timer callback so it would be more accurate.

System Overview

Below is a system overview that outlines the interactions between F-tasks, queues, timers, and DDS functions.

The scheduler itself at a high level is pretty simple. It reads messages from a queue and performs an action based on the type of message. This is shown in the following flow chart. 

In the following sections, each of the five DDS functions are explained. 

Create Task

A software timer periodically creates a new instance of a task. It then calls create_dd_task() and sends it the information about the newly created task. create_dd_task() bundles this information into a dd_task struct and sends it to the scheduler via the messenger queue. The scheduler then adds the new task to active_task_list in the correct order. Below is a diagram of how this works.

As part of creating tasks, our Scheduling task sorts the new task into the correct position. This is the only place where we need to sort the tasks. Below is a simple diagram of the sorting algorithm. 

Complete task

Each DD-task internally keeps track of how long it has been executing. Since a DD-task is only able to communicate with the schedulers auxiliary functions, when the full execution time has occurred, it will call the scheduler function complete_dd_task. This function will then send a message to the scheduler through the messenger queue. Once the scheduler receives this message, it will remove the task from the active_task_list and add it to the completed_task_list to reflect the newly completed task. It will then use vTaskDelete() to delete the F-task.
Get Active Tasks

The monitor task periodically calls get_active_tasks(). This function then sends a message requesting the head of the active_task_list to the scheduler through the messenger queue. Once the scheduler receives this message, it will send the head of the active_task_list to the active_task_queue, which will then be received by get_active_tasks(). Then, the function will return the head of the list to the monitor task.
Get Completed Tasks

The monitor task periodically calls get_completed_tasks(). This function then sends a message requesting the head of the completed_task_list to the scheduler through the messenger queue. Once the scheduler receives this message, it will send the head of the completed_task_list to the completed_task_queue, which will then be received by get_completed_tasks(). Then, the function will return the head of the list to the monitor task.
Get Overdue Tasks

The monitor task periodically calls get_overdue_tasks(). This function then sends a message requesting the head of the overdue_task_list to the scheduler through the messenger queue. Once the scheduler receives this message, it will send the head of the overdue_task_list to the overdue_task_queue, which will then be received by get_overdue_tasks(). Then, the function will return the head of the list to the monitor task.

Discussion

Overall, our scheduler matched the expected performance with an slight error of a few ms. This is likely due to the additional overhead of the monitor task. Below are the test bench results.

Limitations and Possible Improvements
Looking back at this project, we could have made the development process easier by gaining a better understanding of the theory and documentation behind FreeRTOS before starting development. Although we did create a design document at the start of the project, we had a poor understanding of how some FreeRTOS components worked, which made it difficult to plan properly. We did eventually learn all of the necessary documentation as we worked our way through the process, but it would have been easier to do this at the beginning.

Summary

The purpose of this project was to manually create a task scheduling system using FreeRTOS tasks, timers, and queues. The contents of this project were similar to the contents of what we learned in the lecture portion of the class. This made both sections easier, as we learned necessary information in the lecture section, and then applied the information to this lab project.
We had some initial setbacks involving memory management. It was difficult to properly manage pointers as they were constantly getting passed to different functions through queues. This just took some time to figure out, and we were able to make everything functional through basic debugging techniques.
We were able to complete the project with some extra time left, which allowed us to fine tune our task scheduling system, compare our results to the theoretical results, and ensure everything was working correctly.
