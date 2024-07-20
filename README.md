In your React functional component `Posts`, you're correctly initializing a Web Worker (`worker`) and handling the communication between the main thread (React component) and the worker thread (`sort.worker.js`). Here’s how you can pass posts to the worker for sorting:

### Passing Posts to Web Worker

Within the `useEffect` hook where you initialize the worker, you need to send the `posts` array to the worker for sorting. Here’s an updated version of your `Posts` component with comments explaining the process:

```javascript
import React, { useState, useEffect } from 'react';

const Posts = (props) => {
    const [posts, setPosts] = useState(props.posts);
    let worker;

    useEffect(() => {
        // Initialize the Web Worker
        worker = new Worker('sort.worker.js');

        // Event listener to receive messages from the worker
        worker.addEventListener('message', event => {
            const sortedPosts = event.data;
            // Update state with sorted posts
            setPosts(sortedPosts);
        });

        // Clean up function to terminate the worker when component unmounts
        return () => {
            worker.terminate();
        };
    }, []);

    // Function to trigger sorting by sending posts to the worker
    const doSortingByComment = () => {
        if (posts && posts.length) {
            // Send posts array to the worker
            worker.postMessage(posts);
        }
    };

    return (
        <React.Fragment>
            <button onClick={doSortingByComment}>
                Sort By Comments
            </button>
            {/* Render the PostList component with the sorted posts */}
            <PostList posts={posts}></PostList>
        </React.Fragment>
    );
};

export default Posts;
```

### Explanation:

1. **Initialization of Web Worker (`useEffect`)**:
   - In the `useEffect` hook, `worker = new Worker('sort.worker.js');` initializes the Web Worker by specifying the path to `sort.worker.js`.

2. **Event Listener (`addEventListener`)**:
   - `worker.addEventListener('message', event => { ... })` listens for messages from the Web Worker. When the worker sends back the sorted posts (`event.data`), it updates the component's state (`setPosts(sortedPosts)`).

3. **Cleaning Up (`return () => { ... }`)**:
   - `return () => { worker.terminate(); }` ensures the Web Worker is terminated when the component unmounts. This is essential to prevent memory leaks and ensure proper cleanup.

4. **Sorting Trigger (`doSortingByComment`)**:
   - `doSortingByComment` function is called when the "Sort By Comments" button is clicked. It checks if `posts` exists and has elements (`posts && posts.length`), then sends the `posts` array to the Web Worker using `worker.postMessage(posts)`.

5. **Rendering Sorted Posts (`<PostList posts={posts}></PostList>`)**:
   - The `PostList` component (assuming it renders a list of posts) receives the sorted `posts` array as a prop (`posts={posts}`). After sorting, React updates the UI automatically to reflect the sorted order.

### Notes:

- Ensure `sort.worker.js` contains the correct sorting logic and correctly posts back the sorted `posts` array using `self.postMessage(sortedPosts);`.
- Using Web Workers offloads heavy computations to a separate thread, enhancing performance by keeping the main thread (UI thread) responsive.
- This pattern ensures efficient handling of sorting operations while maintaining a clear separation of concerns between the UI logic and background processing.

By following these steps, you effectively pass the `posts` array to the Web Worker for sorting and update your React component’s state with the sorted result upon receiving the sorted data back from the worker.
