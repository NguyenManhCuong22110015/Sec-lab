# Lab #1,22110015, Nguyen Manh Cuong, INSE330380E_01FIE
# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

## 1. Pull docker ubuntu

```bash
docker pull ubuntu
```

## 2. Create 2 computers by ubuntu

```bash
docker run --name sender -it ubuntu
docker run --name receiver -it ubuntu
```

![image](https://github.com/user-attachments/assets/03e8c95b-39f5-444a-8fac-30aa25ea6981)


and after that

```bash
docker run -i receiver
docker run -i sender
```

- finally 

```bash
docker inspect receiver
docker inspect sender
```

to get : ip `sender` : 172.17.0.2 and ip  `receiver`: 172.17.0.1

## 3. Prepare a plaintext file

```bash
echo "THis is plain text file " > plaintext.txt
```

![image](https://github.com/user-attachments/assets/1860f0f3-c86c-4359-9a1f-555cc4e8501c)

## 4. Create a file hash by openssl

- First, dowload openssl
  
```bash
apt install openssl -y
```

- Create a file hash by openssl

```bash
openssl sha256 file.txt > file.txt.hash
```

![image](https://github.com/user-attachments/assets/89d12d40-a230-4be7-91d6-303a94afe9c7)


## 5. Transfering file.txt and file.txt.hash by `Netcat`

- First, dowload netcat

```bash
sudo apt install netcat
```

- In receiver :

```bash
nc -l -p 1234 > plaintext.txt
nc -l -p 1234 > file.txt.hash
```


- In sender: 

```bash
nc 172.17.0.1 1234 < plaintext.txt
nc 172.17.0.1 1234 < file.txt.hash
```

![image](https://github.com/user-attachments/assets/159baf6c-a6e3-4aad-a9ee-77c8db8869a5)

## 6. Veryfing at receiving side.

- Generate the Hash of the Received File 

```bash
openssl sha256 plaintext.txt
```

![image](https://github.com/user-attachments/assets/2997ebf4-f0e6-4ad7-977b-2eda575d444c)

- Open the Sent Hash File

```bash
nano file.txt.hash
```

![image](https://github.com/user-attachments/assets/02abab0c-4483-4e28-a4cc-5fe148c34e42)

The hashes match, confirming that the file is intact and authentic.
 
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:

## 1. Generate RSA Keys (on both computers)

- Each computer needs a pair of RSA keys: a public key (for encryption) and a private key (for decryption).


- In inner-172.16.10.100

```
openssl genpkey -algorithm RSA -out private_key.pem -aes256

openssl rsa -pubout -in private_key.pem -out public_key.pem
```

![image](https://github.com/user-attachments/assets/9442ac14-3fc2-4143-91a4-6f3e3b7d5dc0)


- In outsider-10.9.0.5

```
openssl genpkey -algorithm RSA -out private_key.pem -aes256

openssl rsa -pubout -in private_key.pem -out public_key.pem

```

![image](https://github.com/user-attachments/assets/c56d35fb-cbb3-4caa-a2f3-a5a4369c0683)


## 2. Prepare the File

```
echo "This is a secret message." > file_to_send.txt
```

## 3. Symmetric Encryption (on Sender (inner-172.16.10.100))

- We will symmetrically encrypt the file using a randomly generated secret key. This key will be used to encrypt the file contents. The secret key itself will be encrypted using RSA.

```bash
openssl rand -out secret_key.bin 32

openssl enc -aes-256-cbc -in file_to_send.txt -out file_to_send.txt.enc -pass file:secret_key.bin
```



## 4. Encrypt the Secret Key (on Sender)

- Now we encrypt the secret_key.bin using the public key of Computer B, so only Computer B can decrypt it with its private key.

```bash
openssl rsautl -encrypt -inkey public_key.pem -pubin -in secret_key.bin -out secret_key.bin.enc
```

![image](https://github.com/user-attachments/assets/38bda322-32d1-448a-b067-8a9ef7125401)


## 5.  Transfer the Files

```bash
scp file_to_send.txt.enc secret_key.bin.enc outsider-10.9.0.5@10.9.0.5:/ret
```

## 6. Decrypt the Secret Key (on Receiver)

```bash
openssl rsautl -decrypt -inkey private_key.pem -in secret_key.bin.enc -out secret_key.bin
```

## 7.  Symmetric Decryption of the File (on Receiver)

```bash
openssl enc -d -aes-256-cbc -in file_to_send.txt.enc -out file_to_receive.txt -pass file:secret_key.bin
```

## 8.  Verify the File (on Receiver)

```
bash file_to_send.txt
```
nano 

## 1. 

# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests on the other host.

**Answer 1**:

## 1. Install iptables on both VMs

```bash
sudo apt update
sudo apt install iptables
```

![image](https://github.com/user-attachments/assets/8df3ae26-0188-472a-9aeb-ee62e0030e0a)





