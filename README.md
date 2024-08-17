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
npm install saksh-queue

### Environment Variables
Create a .env file in the root of your project and add your MongoDB connection string:

MONGODB_URL=mongodb://localhost:27017

### Usage

### Basic Setup



This project demonstrates how to use the `saksh-queue` library to manage a queue of data processing tasks using MongoDB and Redlock for distributed locking.

## Installation

To install the required libraries, run:

```bash
npm install saksh-queue
```

### Configuration
Create a .env file in the root directory of your project and add your MongoDB connection URL:

MONGODB_URL=your_mongodb_connection_url

## Usage

### Main Function
The main function initializes the QueueManager, adds data processing tasks to the queue, and processes them.


```
const QueueManager = require('./QueueManager'); // Adjust the path as necessary

// Function to process data and log the result
function sampleFunction(data) {
console.log(`Processing data: ${data}`);
}

async function main() {
const queueManager = new QueueManager();
await queueManager.connect(); // Connect to the queue system

// List of data to process
const dataList = [
'data1',
'data2',
'data3',
// Add more data as needed
];

// Adding tasks with callbacks
for (const data of dataList) {
await queueManager.addTask(
{ data }, 
1, 
0, 
[], 
(task) => {
sampleFunction(task.data); // Use the sample function
}
);
}

// Close the connection after adding tasks
await queueManager.close();
}

main().catch(console.error); // Run the main function and catch any errors



```


### Worker Script
The worker script continuously processes tasks from the queue.



```

const QueueManager = require('./QueueManager'); // Adjust the path as necessary

async function startWorker() {
const queueManager = new QueueManager();
await queueManager.connect(); // Connect to the queue system

// Start processing tasks
await queueManager.startProcessing(5000, 5); // Adjust interval and number of workers as needed

// Handle graceful shutdown
process.on('SIGINT', async () => {
console.log('Shutting down worker...');
await queueManager.close();
process.exit();
});
}

startWorker().catch(console.error);


```



### Testing
To test the functionality of the QueueManager, use the following test.js file:


```

const QueueManager = require('./QueueManager'); // Adjust the path as necessary
const { MongoClient } = require('mongodb');
require('dotenv').config();

describe('QueueManager', () => {
let queueManager;
let client;

beforeAll(async () => {
client = new MongoClient(process.env.MONGODB_URL, { useNewUrlParser: true, useUnifiedTopology: true });
await client.connect();
queueManager = new QueueManager();
await queueManager.connect();
});

afterAll(async () => {
await queueManager.close();
await client.close();
});

test('should add a task to the queue', async () => {
const task = { data: 'testData' };
const taskId = await queueManager.addTask(task);
expect(taskId).toBeDefined();

const addedTask = await queueManager.collection.findOne({ _id: taskId });
expect(addedTask).toBeDefined();
expect(addedTask.task).toEqual(task);
expect(addedTask.status).toBe('pending');
});

test('should process a task from the queue', async () => {
const task = { data: 'processData' };
const taskId = await queueManager.addTask(task);

await queueManager.processQueue();

const processedTask = await queueManager.collection.findOne({ _id: taskId });
expect(processedTask).toBeDefined();
expect(processedTask.status).toBe('completed');
});

test('should recover pending tasks', async () => {
const task = { data: 'recoverData' };
const taskId = await queueManager.addTask(task);

await queueManager.collection.updateOne({ _id: taskId }, { $set: { status: 'processing' } });
await queueManager.recoverPendingTasks();

const recoveredTask = await queueManager.collection.findOne({ _id: taskId });
expect(recoveredTask).toBeDefined();
expect(recoveredTask.status).toBe('pending');
});

test('should expire old tasks', async () => {
const task = { data: 'expireData' };
const taskId = await queueManager.addTask(task);

const expiryTime = new Date(Date.now() - 24 * 60 * 60 * 1000);
await queueManager.collection.updateOne({ _id: taskId }, { $set: { createdAt: expiryTime } });

await queueManager.startProcessing(5000, 1);

const expiredTask = await queueManager.collection.findOne({ _id: taskId });
expect(expiredTask).toBeDefined();
expect(expiredTask.status).toBe('expired');
});
});
```


### Running the Tests
To run the tests, add the following script to your package.json:
```
"scripts": {
"test": "jest"
}
```
Then, run the tests using:
```
npm test
```
### Examples
Here are some additional examples of adding tasks to the queue:

Example 1: Adding a Task with Higher Priority

```

await queueManager.addTask(
{ data: 'highpriority' }, // Task data
10, // Higher priority
0, // No delay
[], // No dependencies
(task) => {
sampleFunction(task.data); // Use the sample function
}
);
```

#### Example 2: Adding a Task with Delay

```

await queueManager.addTask(
{ data: 'delayed' }, // Task data
1, // Normal priority
5000, // Delay of 5000 milliseconds (5 seconds)
[], // No dependencies
(task) => {
sampleFunction(task.data); // Use the sample function
}
);
```


#### Example 3: Adding a Task with Dependencies
```

// Assume task1 and task2 are IDs of previously added tasks
const dependencies = ['task1', 'task2'];

await queueManager.addTask(
{ data: 'dependent' }, // Task data
1, // Normal priority
0, // No delay
dependencies, // Dependencies on task1 and task2
(task) => {
sampleFunction(task.data); // Use the sample function
}
);
```


#### Example 4: Adding Multiple Tasks in a Loop

```
const moreData = [
'data4',
'data5',
'data6',
];

for (const data of moreData) {
await queueManager.addTask(
{ data }, 
1, 
0, 
[], 
(task) => {
sampleFunction(task.data); // Use the sample function
}
);
}

```



#### Example 5: Adding a Task with a Custom Callback

```

await queueManager.addTask(
{ data: 'customcallback' }, // Task data
1, // Normal priority
0, // No delay
[], // No dependencies
(task) => {
console.log(`Custom Callback: Processing data: ${task.data}`);
}
);
```


#### Example 6: Adding a Task with Low Priority

```
await queueManager.addTask(
{ data: 'lowpriority' }, // Task data
0, // Low priority
0, // No delay
[], // No dependencies
(task) => {
sampleFunction(task.data); // Use the sample function
}
);

```

#### Example 7: Adding a Task with a Long Delay

```

await queueManager.addTask(
{ data: 'longdelay' }, // Task data
1, // Normal priority
60000, // Delay of 60000 milliseconds (1 minute)
[], // No dependencies
(task) => {
sampleFunction(task.data); // Use the sample function
}
);

```



#### Example 8: Adding a Task with Multiple Dependencies

```


// Assume task3 and task4 are IDs of previously added tasks
const multipleDependencies = ['task3', 'task4'];

await queueManager.addTask(
{ data: 'multipledependent' }, // Task data
1, // Normal priority
0, // No delay
multipleDependencies, // Dependencies on task3 and task4
(task) => {
sampleFunction(task.data); // Use the sample function
}
);

```

#### Example 9: Adding a Task with Immediate Execution

```
await queueManager.addTask(
{ data: 'immediate' }, // Task data
1, // Normal priority
0, // No delay
[], // No dependencies
(task) => {
sampleFunction(task.data); // Use the sample function
}
);
 ```


#### Example 10: Adding a Task with a Different Callback


```
await queueManager.addTask(
{ data: 'differentcallback' }, // Task data
1, // Normal priority
0, // No delay
[], // No dependencies
(task) => {
console.log(`Different Callback: Processing data: ${task.data}`);
}
);
```


#### Explanation
•  Task Data: { data: 'differentcallback' } - This is the data associated with the task.

•  Priority: 1 - This sets the task's priority to normal.

•  Delay: 0 - This means there is no delay before the task is processed.

•  Dependencies: [] - This task has no dependencies.

•  Callback Function: (task) => { console.log(Different Callback: Processing data: ${task.data}); } - This is a custom callback function that logs a different message when processing the task.

This example shows how you can customize the callback function for each task, allowing you to handle different types of tasks in various ways.

If you have any more questions or need further assistance, feel free to ask!






#### License
This project is licensed under the MIT License.

### Support
contact susheel2339 at gmail dot come

 
