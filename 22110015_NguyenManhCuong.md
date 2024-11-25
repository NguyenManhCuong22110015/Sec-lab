# Lab #1,22110015, Nguyen Manh Cuong, INSE330380E_01FIE
# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

## 1. Set up 2 computers in docker 

```bash
https://github.com/quang-ute/Security-labs
```

and run docker in  

```bash
cd Network/firewall
docker-compose up -d
```

![image](https://github.com/user-attachments/assets/1922078c-1684-44f3-9955-205fcf3b112e)

## 2. Use 2 computers

```bash
docker exec -it  --user root inner-172.16.10.100 sh -l
docker exec -it  --user root outsider-10.9.0.5 sh -l
```

## 3. Create a plaintext file named `file.txt`
* First, we write a message and save it in a text file:
```bash
echo "This is a test file." > file.txt
```

## 4. Create a file hash by `openssl`

```bash
openssl sha256 file.txt > file.txt.hash
```
![image](https://github.com/user-attachments/assets/3a8d2523-ebd7-4c9f-ae76-b87c83c3c980)


## 5. Transfering `file.txt` and `file.txt.hash`

```bash
 docker cp inner-172.16.10.100:/tsec-lab/file.txt ./file.tx
 docker cp inner-172.16.10.100:/tsec-lab/file.txt.hash ./file.tx.hash
```
![image](https://github.com/user-attachments/assets/10fa66d5-c3e1-46b6-b222-1b2584ac01e1)
![image](https://github.com/user-attachments/assets/c98f1d7b-029a-49de-8d06-42d16d169571)


```bash
 docker cp file.tx outsider-10.9.0.5:/res/file.txt
 docker cp file.tx.hash outsider-10.9.0.5:/res/file.txt.hash
```
![image](https://github.com/user-attachments/assets/2dc2584f-01dd-4f86-bef2-4a47881bf1c8)

- Check in `outsider-10.9.0.5:/res`
- 
![image](https://github.com/user-attachments/assets/5daf53b8-722f-487c-8d67-5af18d240765)

## 6.  Veryfing at receiving side

```
openssl sha256 file.txt > check.txt.hash
```

![image](https://github.com/user-attachments/assets/0d0908c0-5057-4797-8c28-6e963adafe49)

 If the hashes of the original and received files are the same, you can trust that the file has not been tampered with during transfer and is intact.
 
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

