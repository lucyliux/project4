# High-level approach:
This program implements a TCP-like transport protocol. 

The sender reads data from stdin, and sends it to the receiver along with other fields such as type of data, sequence number and hash of data in a single message through a UDP socket. Meanwhile, it also receives acknowledgements of sent packets from the receiver. There is a congestion window (cwnd) that limits the number of packets that the sender is able to send before getting ACKs from the receiver. It grows slowly and adjusts to the bandwidth. If the sender receives 3 duplicate ACKs from the receiver, the sender retransmits all sent packets that are waiting for ACKs. There is a RTT, which represents the round trip time for a packet to travel between sender and receiver. RTT is initialized to 0.5 at first, and gets recalculated everytime the sender receives an ACK or experiences timeout, so that the program keeps up with the constantly changing network latency. In every outgoing packet to the receiver, the sender generates a MD5 hash for the data and sends it together, so that the receiver can compare the received hash with hash of data to detect packet corruption. It runs until there is no data coming from stdin and all sent packets have been acknowledged and gracefully exits.

On the receiver side, it receives data packets from the receiver through a UDP socket. On receiving the packet, the receiver first check if the packet is corrupted by comparing the received hash and the hash of received data. If they don't match, the receiver treats it as if it is a lost packet (i.e. do nothing). If the packet is not corrupted, the receiver checks if it's in order, out of order or a duplicate. If it is an in order packet, it sends an ACK to the sender and prints it to stdout immediately. If it is an out of order packet (meaning that there are packets that have smaller sequence numbers that haven't been received), it gets stored to a buffer list, and when all packets with smaller sequence numbers are received, the receiver prints it to stdout. If it is a duplicate packet, we simply send an ACK to the sender. ACKs are sent upon all non-corrupted message arrivals. 

# Challenges faced:
The major challenge we faced was deciding which implementation to use. We started off with very simple algorithms, hoping it would be less error prone, but it turned out that those algorithms do not cover all possible cases (e.g. cannot fix certain packet loss). After discussing and implementing various algorithms together, and reading through the slides, we finally decide to follow the pseudocode in the slides and it worked well.

# Design highlights
One of our design highlight is that in the sender we have two different lists for packets loaded from stdin and packets that need to be retransmitted. Similarly, we also have a list in the receiver that stores out of order packets that are received. This makes makes it easier to determine the state of the packet and do operations correspondingly.

# Testing Strategy:
To make sure our code behaves correctly, we print debugging messages to stderr when:

Sender:
- sends a packet: prints a sending message that containd sequence number and block number
- receives an ACK: prints the sequence number of that ACK
- receives duplicate ACK: prints the number of duplicate ACK
- retransmitting due to 3 duplicate ACKs or timeout
- constantly prints the sequence number that are waiting to be acknowledged

Receiver:
- received a packet: prints the sequence number and block number
- sends an ACK: prints the sequence number
- prints a message to stdout: prints the sequence number
- received a duplicate packet: prints the sequence number and block number

Being able to see these messages when the code is running is extremely helpful as we can know what packets are being sent, what packets are being lost, and what packets the sender and receiver are waiting on.
