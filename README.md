# SC6103_DS: A Distributed Flight Information System
 - CS Architecture
 - UDP sockets
![CS Communication Flow](cs-communication-flow.png)

## testInternet: Stonesama!!!!!
## Three computers are in the same network
## Close the firewall on the server
## Server

### store the information of all flights
```
flight:
{
    flight_id: int
    source_place: variable-length str
    destination_place: variable-length str
    departure_time: {
        year: int
        month: int
        day: int
        hour: int
        minute: int
    }
    airfare: float
    seat_availability: int // num of seats available
    baggage_availability: int
}
```

Marshalling (normal) → byte
byte Message (marshlled) 
socketpackage (massa)
unmarshalling (byte) → normal



### implement services on the flights for remote access by clients
```
1. query_flight_id (source_place, destination_place) {
    if multiple flights match:
        return a list of all
    if no flight matches:
        return an error message
}

2. query_flight_info (flight_id) {
    if flight_id does not exist:
        return an error message
    else:
        return departure_time, airfare, seat_availability
}

3. make_seat_reservation (flight_id, num_seats) {
    if successful reservation:
        return acknowledgement to client
        update seat_availability on server 
    if flight_id does not exist or insufficient available for num_seats:
        return an error message
}

4. [idempotent] query_baggage_availability (flight_id) {
    if flight_id does not exist:
        return an error message
    else:
        return baggage_availability
}

5. [non-idempotent] add_baggage (flight_id, num_baggages) {
    if successful reservation:
        return acknowledgement to client
        update baggage_availability on server
    if flight_id does not exist or insufficient available for num_baggages:
        return an error message
}

6. callback
    client:
        public interface Callback extends Remote {
            void update_seat_availability (flight_id, seat_availability) throws RemoteException;
        }
    server: 多线程能够处理多个client的同时更新
        public interface Monitor extends Remote {
            void register (Callback cbObject, flight_id, monitor_interval) throws RemoteException {
                record the server_address & port_number;
                if monitor_interval expire then remove the client record;
            }
            void deregister (Callback cbObject) throws RemoteException;
        }


7. create a new thread to serve each request received, and record the client address & port when it receives the request.

8. The received requests and the produced replies should be
printed on the screen.
```

## Client
1. provide an interface that repeatedly asks the user to enter a request and sends the request to the server (client & user - rolling)
 
2. include an option for the user to terminate the client

3. already know the server address and port number

4. Each reply or error message returned from the server should be printed on the screen.

5. GUI


## Message
1. request-reply message structure
    - message_type: int, 1 Byte
        - 0xxx xxxx request
            - 0xxx 0 register
            - 0xxx 1 query_flight_id
            - 0xxx 2 query_flight_info
            - 0xxx 3 make_seat_reservation
            - 0xxx 4 query_baggage_availability
            - 0xxx 5 add_baggage
        - 1xxx xxxx reply 同上顺序
    - request_id: int, 4 Bytes
        - client_id
            - client_addr
            - client_port
        - unique_identifier
    - data_length: int, 4 Bytes
    - data: flight
        - int: 4 Bytes
        - float: 4 Bytes
        - variable-length str
            - str_length: 4 Bytes
            - str_content: 1 Byte for 1 character
2. marshaling & unmarshalling
```
 #include <netinet/in.h>
 uint32_t htonl(uint32_t hostlong);
 uint32_t ntohl(uint32_t netlong);
```
3. fault-tolerance
    - at-least-once
        - server: re-execute
        - client: timeout
    - at-most-once
        - server: store history + filter duplicate + re-reply
        - client: timeout
4. Which semantics to use can be specified as an argument in the command that starts the client/server



# Meeting Notes
## 2024.10.06 19:00
 1. due
    - 2024.10.16 code + report
    - 2024.10.19 demonstration
 2. task decomposition and distribution
    - server side (C) - Gaohan & Ziling
    - client side (Java) - Fanhui & Shuangyue
 3. customized function determination
    - idempotent: choose meal
    - non-idempotent: buy VIP lounge or other additional services

## 2024.10.12 after DS class



## Database
	CREATE DATABASE flight_system;
 ![image](https://github.com/user-attachments/assets/854a87a7-0e3b-49f4-be92-21ceb1656626)
 
	USE flight_system;
 ![image](https://github.com/user-attachments/assets/7d17ed79-2d3c-47f4-9caf-17a4f49a7200)

 	CREATE TABLE flights (
	flight_id INT AUTO_INCREMENT PRIMARY KEY,
	source_place VARCHAR(100) NOT NULL,
	destination_place VARCHAR(100) NOT NULL,
	departure_year INT NOT NULL,
	departure_month INT NOT NULL,
	departure_day INT NOT NULL,
	departure_hour INT NOT NULL,
	departure_minute INT NOT NULL,
	airfare FLOAT NOT NULL,
	seat_availability INT NOT NULL,
	baggage_availability INT NOT NULL
 	);
![image](https://github.com/user-attachments/assets/eac83249-68d0-4d1b-b045-011e88e62df7)

	INSERT INTO flights (source_place, destination_place, departure_year, departure_month, departure_day, departure_hour, departure_minute, airfare, 	seat_availability, baggage_availability)
	VALUES 
	('Singapore', 'Tokyo', 2024, 10, 12, 8, 0, 500.0, 50, 100),
	('Singapore', 'New York', 2024, 10, 13, 23, 0, 1200.0, 30, 50);
![image](https://github.com/user-attachments/assets/15955ded-6f2c-491c-b7c2-fa2857e98022)

	SELECT * FROM flights;
![image](https://github.com/user-attachments/assets/df31c795-4781-4066-a20d-d8b23099c47f)

	gcc -o database_connect F:/ntu/course/6103/SC6103_DS/src/server_c/database_connect.c -IF:/anzhuangbao/Wnmp-4.2.0/mariadb-bins/default/include -	LF:/anzhuangbao/Wnmp-4.2.0/mariadb-bins/default/lib F:/anzhuangbao/Wnmp-4.2.0/mariadb-bins/default/lib/libmariadb.dll
![image](https://github.com/user-attachments/assets/1f541da9-b92d-47f0-8601-bf052a54a642)

	F:/ntu/course/6103/SC6103_DS/src/server_c/database_connect.exe
![image](https://github.com/user-attachments/assets/4fac1f70-e702-4880-b7eb-1c40608edbd4)

### Fault tolerance strategy selection:

The use_at_least_once flag controls the fault tolerance strategy. When starting the server from the command line, you can choose to use at-least-once or at-most-once:

./server at-least-once

./server at-most-once

The server decides which fault tolerance mechanism to use based on the parameters passed in.

### at-least-once mechanism:
Each request is re-executed, regardless of whether the request is repeated or not.
### at-most-once mechanism:
Each request first checks the history to avoid repeated execution. If it is found that the request has been processed, the cached response is returned directly.

### Multithreading support:
Every time a request is received, a new thread is created to handle the client request.

### How to use:
Compile and run the server:

gcc server.c callback_handler.c data_storage.c flight_service.c thread_pool.c marshalling.c unmarshalling.c handleRequest.c database_connect.c -o server -lpthread -lws2_32 -IF:/anzhuangbao/Wnmp-4.2.0/mariadb-bins/default/include -LF:/anzhuangbao/Wnmp-4.2.0/mariadb-bins/default/lib F:/anzhuangbao/Wnmp-4.2.0/mariadb-bins/default/lib/libmariadb.dll

Run the server and select the fault tolerance mechanism:

./server at-least-once # Use the at-least-once mechanism

./server at-most-once # Use the at-most-once mechanism

### Summary:
This code allows the server to switch between different fault tolerance mechanisms at startup based on the parameters selected by the user. With the at-least-once mechanism, the server will re-execute the request each time, while with the at-most-once mechanism, the server will avoid processing duplicate requests.

**1. Introduction to Network Interface Programming**

- Definition and purpose of interface
- Overview of interface communication (client-server model)
- Concept of inter-process communication

**2. Overview of interfaces and ports**

- Description of interface terminals
- Pairing to bind an interface to (network address, local port)
- Details of inter-process communication

**3. Interface programming in C language**

- Overview of interface programming on UNIX system
- Main system calls:
- `socket()`: create interface
- `close()`: destroy interface
- `bind()`: bind interface to address
- `sendto()` and `recvfrom()`: send and receive data
- `select()`: detect arrived messages
- Example of implementing UDP communication in C language:
- Client-server message exchange

**4. TCP communication in C language**

- System calls for TCP communication
- Establish connection (`listen()`, `accept()` on server side and `connect()` on client side)
- Use `write()` and `read()` for data transmission
- Example of TCP client-server interaction



To build the server, please follow the steps below:

### 1. Project structure setup
Create folders and files according to the given project structure:
```bash
mkdir -p flight_booking_system/server
cd flight_booking_system/server
touch server.c flight_service.c flight_service.h callback_handler.c callback_handler.h data_storage.c data_storage.h thread_pool.c thread_pool.h Makefile
```
### 2. Write code
Refine the code according to the provided framework. Implement the corresponding functions in each file:

- **server.c**: The main server program, responsible for listening to client requests and processing them.
- **flight_service.c / flight_service.h**: Implement services such as flight query, flight details and seat reservation.
- **callback_handler.c / callback_handler.h**: Used to handle callback requests in client monitoring services.
- **data_storage.c / data_storage.h**: Responsible for the management of flight data (storage, addition, update, etc.).
- **thread_pool.c / thread_pool.h**: Provides a simple thread pool for handling concurrent requests.

#### server.c (main program framework)
Mainly implements the creation and binding of server sockets, as well as the logic of request processing.

#### flight_service.c / flight_service.h
All flight service related functions are implemented here, such as flight query, flight details, seat reservation, etc. Migrate the flight processing code in `server.c` to `flight_service.c`, and use the header file to declare these functions so that they can be called in the main program.

For example:
```c
// flight_service.h
#ifndef FLIGHT_SERVICE_H
#define FLIGHT_SERVICE_H

#include <netinet/in.h>

void handle_query_flight(int sockfd, struct sockaddr_in *client_addr, char *buffer);
void handle_query_details(int sockfd, struct sockaddr_in *client_addr, char *buffer);
void handle_reservation(int sockfd, struct sockaddr_in *client_addr, char *buffer);

#endif
```

### 3. Implement Makefile
Create a `Makefile` to manage the compilation process, support compiling all source files and generating executable files:
```makefile
CC = gcc
CFLAGS = -Wall -g
OBJ = server.o flight_service.o callback_handler.o data_storage.o thread_pool.o udp_utils.o

all: server

server: $(OBJ)
	$(CC) $(CFLAGS) -o server $(OBJ)

server.o: server.c
	$(CC) $(CFLAGS) -c server.c

flight_service.o: flight_service.c flight_service.h
	$(CC) $(CFLAGS) -c flight_service.c

callback_handler.o: callback_handler.c callback_handler.h
	$(CC) $(CFLAGS) -c callback_handler.c

data_storage.o: data_storage.c data_storage.h
	$(CC) $(CFLAGS) -c data_storage.c

thread_pool.o: thread_pool.c thread_pool.h
	$(CC) $(CFLAGS) -c thread_pool.c

udp_utils.o: ../common/udp_utils.c ../common/udp_utils.h
	$(CC) $(CFLAGS) -c ../common/udp_utils.c

clean:
	rm -f *.o server
```
This Makefile supports compiling the source files of each module and generating the server executable file `server`. You can compile the code by executing the `make` command.

### 4. Compile and run
First, go to the `server` directory and compile using `Makefile`:
```bash
cd flight_booking_system/server
make
```
If everything goes well, you will see the generated executable file `server`.

Then run the server:
```bash
./server
```
This will start the UDP server and listen for client requests on port 8080.

### 5. Test
You can use the client program in the `client_java` project to test whether the server is working properly. Make sure to set the correct server IP and port in `client_java/resources/config.properties`.

### Summary
Through the above steps, the server-side project has been built, and the division of labor and cooperation between modules has been realized. This modular design is helpful for development and maintenance. For example, you can independently develop and test the flight service module, callback processing module, etc. If you encounter any problems or need further explanation, please let me know.



