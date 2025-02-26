
## **Kafka on AWS EC2 Instance**

### **Step 1: Launch an EC2 Instance**
1. **Log in to AWS Console**: Go to the [AWS Management Console](https://aws.amazon.com/console/).
2. **Launch EC2 Instance**:
   - Navigate to **EC2** > **Instances** > **Launch Instance**.
   - Choose an **Amazon Linux 2 AMI** (free tier eligible).
   - Select the **t2.micro** instance type (free tier eligible).
   - Configure instance details (default settings are fine for this lab).
   - Add storage (default 8GB is sufficient).
   - Add tags (optional).
   - Configure security group:
     - Add rules to allow **SSH (port 22)** and **Kafka (port 9092)**.
     - Example:
       - Type: SSH, Protocol: TCP, Port Range: 22, Source: My IP.
       - Type: Custom TCP, Protocol: TCP, Port Range: 9092, Source: Anywhere (0.0.0.0/0).
   - Review and launch the instance.
   - Create or use an existing key pair to connect to the instance.

---

### **Step 2: Connect to the EC2 Instance**
1. **SSH into the Instance**:
   - Use the private key (.pem file) to connect to the instance.
   - Example command:
     ```bash
     ssh -i /path/to/your-key.pem ec2-user@<Public-IP-of-EC2>
     ```

---

### **Step 3: Install Java**
Kafka requires Java to run. Install Java on the EC2 instance:
```bash
sudo yum update -y
sudo yum install java-1.8.0-openjdk -y
```
Verify the installation:
```bash
java -version
```

---

### **Step 4: Download and Install Apache Kafka**
1. **Download Kafka**:
   - Download the latest Kafka binary from the [Apache Kafka website](https://kafka.apache.org/downloads).
   - Example:
     ```bash
     wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
     ```
2. **Extract Kafka**:
   ```bash
   tar -xzf kafka_2.13-3.6.0.tgz
   ```
3. **Move Kafka to a convenient location**:
   ```bash
   mv kafka_2.13-3.6.0 /opt/kafka
   ```

---

### **Step 5: Configure Kafka**
1. **Edit Kafka Configuration**:
   - Open the server properties file:
     ```bash
     sudo nano /opt/kafka/config/server.properties
     ```
   - Update the following properties:
     ```properties
     listeners=PLAINTEXT://<Private-IP-of-EC2>:9092
     advertised.listeners=PLAINTEXT://<Public-IP-of-EC2>:9092
     ```
   - Save and exit the file.

2. **Edit Zookeeper Configuration** (if using standalone Zookeeper):
   - Open the Zookeeper properties file:
     ```bash
     sudo nano /opt/kafka/config/zookeeper.properties
     ```
   - No changes are required for a basic setup.

---

### **Step 6: Start Zookeeper and Kafka**
1. **Start Zookeeper**:
   ```bash
   /opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties &
   ```
2. **Start Kafka**:
   ```bash
   /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &
   ```

---

### **Step 7: Create a Kafka Topic**
1. **Create a Topic**:
   ```bash
   /opt/kafka/bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
   ```
2. **List Topics**:
   ```bash
   /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
   ```

---

### **Step 8: Produce and Consume Messages**
1. **Start a Producer**:
   - Open a new terminal and SSH into the EC2 instance.
   - Run the producer:
     ```bash
     /opt/kafka/bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
     ```
   - Type messages in the terminal and press Enter to send them to the topic.

2. **Start a Consumer**:
   - Open another terminal and SSH into the EC2 instance.
   - Run the consumer:
     ```bash
     /opt/kafka/bin/kafka-console-consumer.sh --topic test-topic --bootstrap-server localhost:9092 --from-beginning
     ```
   - You should see the messages produced by the producer.

---

### **Step 9: Integrate with a Simple Data Pipeline**
1. **Simulate a Data Source**:
   - Create a script to generate sample data and send it to Kafka.
   - Example Python script (`producer.py`):
     ```python
     from kafka import KafkaProducer
     import time

     producer = KafkaProducer(bootstrap_servers='<Public-IP-of-EC2>:9092')

     for i in range(10):
         message = f"Message {i}"
         producer.send('test-topic', message.encode('utf-8'))
         print(f"Sent: {message}")
         time.sleep(1)

     producer.flush()
     ```

2. **Simulate a Data Sink**:
   - Create a script to consume data from Kafka and process it.
   - Example Python script (`consumer.py`):
     ```python
     from kafka import KafkaConsumer

     consumer = KafkaConsumer('test-topic', bootstrap_servers='<Public-IP-of-EC2>:9092')

     for message in consumer:
         print(f"Received: {message.value.decode('utf-8')}")
     ```

3. **Run the Scripts**:
   - Install the Kafka Python library:
     ```bash
     pip install kafka-python
     ```
   - Run the producer and consumer scripts in separate terminals.

---

### **Step 10: Clean Up**
1. **Stop Kafka and Zookeeper**:
   ```bash
   /opt/kafka/bin/kafka-server-stop.sh
   /opt/kafka/bin/zookeeper-server-stop.sh
   ```
2. **Terminate the EC2 Instance**:
   - Go to the AWS Console > EC2 > Instances.
   - Select the instance and click **Terminate**.

---
