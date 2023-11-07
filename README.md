# System Programming Assignment - Data Server & Remote Client

## Overview
This project entails the complete implementation of a data server and a remote client. The data server initializes and synchronizes threads and structures to serve client requests, while the client is a simple program designed to connect to the server and request files.

### Server Operation
#### SERVER
1. The server starts and reads command-line arguments.
2. Initializes the necessary mutexes for thread synchronization and other structures and variables.
3. Creates a listening socket to accept new client connections.
4. Initializes worker threads responsible for sending files to clients. These threads are given an index argument that corresponds to their position in the 'busy' array, which tracks whether a socket is occupied by a thread.
5. Enters an infinite loop, accepting new connections and creating new communication threads that read the requested files and place them in a custom queue.

#### COMMUNICATION THREAD
When the server accepts a new connection, it creates a new communication thread:
1. Sends the block_size to the client and receives the requested path.
2. Uses a custom fpath class to create a structure of file information (path, whether it's a folder, name, etc.) without storing file data.
3. Finds the number of files in the requested path.
4. Maintains a vector recording the form [current_socket, number_of_files, files_served], managing which sockets have been fully served to safely close connections.
5. Adds q_nodes to the queue, which activates workers via mutexes to start serving clients.

#### WORKER THREAD
Worker threads wait in an endless loop for a node to enter the shared queue. Once activated:
1. Takes control of the queue with mutexes.
2. Retrieves the first element from the queue.
3. Checks if the associated socket is busy.
4. If the socket is busy, moves the node to the back of the queue and repeats step 2.
5. If the socket is not busy, marks it as busy in the 'busy' array at the given index.
6. Sends file information and then the file itself to the client in block_size chunks.
7. Increments the 'files served' counter and closes the connection when all files are served.

The above process repeats continuously.
