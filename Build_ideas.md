## Require
+ design and implement a distributed flight 
+ client-server architecture
+  information of all flights is stored at the server
+ variable
	+  flight identifier (integer)
	+  the source and destination places (variable-length strings)
	+  the departure time (design your own data structure to represent time)
	+  the airfare (floating-point value)
	+  the seat availability (an integer indicating the number of seats available)
+ transport protocol： UDP
+ Server
	+ The server program implements a set of services on the flights for remote access by clients. 
	 + The server performs the requested service and returns the result to the client.
+ Client
	+ the client program provides an interface for users to invoke these services.
	+ On receiving a request input from the user, the client sends the request to the server
	+ The client then presents the result on the console to the user
	
+ refer to the online notes on socket programming 

+ Server Services
 be implemented by the server program and made available to the users through the client program
	1. query_flight_id (source_place, destination_place) {
    if multiple flights match:
        return a list of all
    if no flight matches:
        return an error message
}

	2. query_departure_time (flight_id)
    query_airfare (flight_id)
    query_seat_availability (flight_id)
    if flight_id does not exist:
        return an error message

	3. make_seat_reservation (flight_id, num_seats) {
    		if successful reservation:
        		return acknowledgement to client
        		update seat_availability on server 
    		if incorrect user input (flight_id does not exist or insufficient available for num_seats):
        		return an error message
}

	4. callback (server & client): monitor_seat_availability (main pre content)
	5. two more operations on the flights through client-server communication:
    		- one idempotent 选餐
    		- one non-idempotent VIP休息室+其他各类需求购买
	6. create a new thread to serve each request received

+ Client Services
	1. provide an interface that repeatedly asks the user to enter a request and sends the request to the server

	2. include an option for the user to terminate the client

	3. already know the server address and port number

	4. Message

		+ self-design format
		+ transmit in byte array
			+ marshaling (int / float / str)
			+ unmarshalling
 ```#include <netinet/in.h>
 uint32_t htonl(uint32_t hostlong);
 uint32_t ntohl(uint32_t netlong);```
