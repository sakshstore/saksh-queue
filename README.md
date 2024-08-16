## Saksh Queue  
A simple and scalable queue management system using Node.js and MongoDB, with support for task priorities, dependencies, error handling, notifications, persistence, recovery, and horizontal scalability.

### Features
1. Task Prioritization: Assign priorities to tasks to control the order of execution.
2. Task Delays: Schedule tasks to be processed after a certain delay.
3. Task Dependencies: Ensure tasks are processed only after their dependencies are completed.
4. Error Handling and Notifications: Handle errors gracefully and notify via a callback function.
5. Persistence and Recovery: Recover from failures and persist the queue state across restarts.
6. Scalability: Scale the queue management system horizontally, allowing multiple instances to work together.

### Installation
npm install queue-management-package

### Environment Variables
Create a .env file in the root of your project and add your MongoDB connection string:

MONGODB_URL=mongodb://localhost:27017

### Usage

### Basic Setup

```
const QueueManager = require('saksh-queue');

async function main() {
const queueManager = new QueueManager();
await queueManager.connect();

// Adding tasks with different priorities
await queueManager.addTask('Task A', 1); // Priority 1
await queueManager.addTask('Task B', 2); // Priority 2
await queueManager.addTask('Task C', 3); // Priority 3

const processTask = async (task) => {
console.log('Processing:', task);
await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate task processing
};

const errorCallback = (task, error) => {
console.error(`Error processing task "${task}":`, error);
};

queueManager.startProcessing(processTask, errorCallback);

// Close the connection after some time
setTimeout(() => queueManager.close(), 60000);
}

main().catch(console.error);
```


### Task with Dependencies

```
const QueueManager = require('queue-management-package');

async function main() {
const queueManager = new QueueManager();
await queueManager.connect();

// Adding tasks with dependencies
const task1Id = await queueManager.addTask('Task 1', 1); // Priority 1
const task2Id = await queueManager.addTask('Task 2', 2, 0, [task1Id]); // Priority 2, dependent on Task 1
const task3Id = await queueManager.addTask('Task 3', 3, 0, [task2Id]); // Priority 3, dependent on Task 2

const processTask = async (task) => {
console.log('Processing:', task);
await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate task processing
};

const errorCallback = (task, error) => {
console.error(`Error processing task "${task}":`, error);
};

queueManager.startProcessing(processTask, errorCallback);

// Close the connection after some time
setTimeout(() => queueManager.close(), 60000);
}

main().catch(console.error);

```


#### Handling Errors and Notifications

```
const QueueManager = require('queue-management-package');

async function main() {
const queueManager = new QueueManager();
await queueManager.connect();

// Adding tasks
await queueManager.addTask('Task X', 1); // Priority 1
await queueManager.addTask('Task Y', 2); // Priority 2

const processTask = async (task) => {
console.log('Processing:', task);
if (task === 'Task X') {
throw new Error('Simulated error'); // Simulate an error for Task X
}
await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate task processing
};

const errorCallback = (task, error) => {
console.error(`Error processing task "${task}":`, error);
// Custom error handling logic, e.g., send an email or log to a monitoring system
};

queueManager.startProcessing(processTask, errorCallback);

// Close the connection after some time
setTimeout(() => queueManager.close(), 60000);
}

main().catch(console.error);
```


#### Persistence and Recovery
```
const QueueManager = require('queue-management-package');

async function main() {
const queueManager = new QueueManager();
await queueManager.connect();

// Adding tasks
await queueManager.addTask('Task 1', 1); // Priority 1
await queueManager.addTask('Task 2', 2); // Priority 2

const processTask = async (task) => {
console.log('Processing:', task);
await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate task processing
};

const errorCallback = (task, error) => {
console.error(`Error processing task "${task}":`, error);
};

queueManager.startProcessing(processTask, errorCallback);

// Simulate a restart by closing and reconnecting
setTimeout(async () => {
await queueManager.close();
await queueManager.connect();
queueManager.startProcessing(processTask, errorCallback);
}, 30000);

// Close the connection after some time
setTimeout(() => queueManager.close(), 90000);
}

main().catch(console.error);


```



#### Scalability with Multiple Instances

```
const QueueManager = require('queue-management-package');

async function main() {
const queueManager1 = new QueueManager();
const queueManager2 = new QueueManager();
await queueManager1.connect();
await queueManager2.connect();

// Adding tasks
await queueManager1.addTask('Task 1', 1); // Priority 1
await queueManager1.addTask('Task 2', 2); // Priority 2

const processTask = async (task) => {
console.log('Processing:', task);
await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate task processing
};

const errorCallback = (task, error) => {
console.error(`Error processing task "${task}":`, error);
};

queueManager1.startProcessing(processTask, errorCallback);
queueManager2.startProcessing(processTask, errorCallback);

// Close the connections after some time
setTimeout(() => {
queueManager1.close();
queueManager2.close();
}, 60000);
}

main().catch(console.error);


```
#### License
This project is licensed under the MIT License.

### Support
contact susheel2339 at gmail dot come


Feel free to customize this README document further to suit your specific needs. Let me know if you need any more examples or additional information!
