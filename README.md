


import socket
import threading
import json

# P2P नोड क्लास
class P2PNode:
    def __init__(self, host='localhost', port=5000):
        self.host = host
        self.port = port
        self.peers = []
        self.blockchain = Blockchain()
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.bind((self.host, self.port))
        self.server.listen(5)
        print(f"Node running on {self.host}:{self.port}")

        # सर्वर थ्रेड शुरू करें
        threading.Thread(target=self.accept_connections, daemon=True).start()

    def accept_connections(self):
        while True:
            client, addr = self.server.accept()
            print(f"Connected to {addr}")
            threading.Thread(target=self.handle_client, args=(client,), daemon=True).start()

    def handle_client(self, client):
        while True:
            try:
                message = client.recv(1024).decode()
                if message:
                    data = json.loads(message)
                    if data['type'] == 'block':
                        self.blockchain.add_block(data['transactions'])
                        self.broadcast_block(data['transactions'])
            except:
                client.close()
                break

    def connect_to_peer(self, peer_host, peer_port):
        peer = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        peer.connect((peer_host, peer_port))
        self.peers.append(peer)
        print(f"Connected to peer {peer_host}:{peer_port}")

    def broadcast_block(self, transactions):
        block_data = json.dumps({"type": "block", "transactions": transactions})
        for peer in self.peers:
            peer.send(block_data.encode())

# टेस्टिंग
if __name__ == "__main__":
    # नोड 1
    node1 = P2PNode('localhost', 5000)

    # नोड 2
    node2 = P2PNode('localhost', 5001)

    # नोड 1 को नोड 2 से कनेक्ट करें
    node1.connect_to_peer('localhost', 5001)

    # नोड 1 पर एक ट्रांजैक्शन जोड़ें
    node1.blockchain.add_block(["Alice sent 10 coins to Bob"])
    node1.broadcast_block(["Alice sent 10 coins to Bob"])

    # थोड़ा इंतजार करें ताकि नोड 2 को डेटा मिल जाए
    time.sleep(2)

    # नोड 2 की चेन प्रिंट करें
    print("Node 2's Blockchain:")
    for block in node2.blockchain.chain:
        print(f"Block #{block.index}")
        print(f"Transactions: {block.transactions}")
        print(f"Hash: {block.hash}\n")
