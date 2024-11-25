# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

## 1. Set up 2 computers in docker 

```bash
docker-compose up -d
```

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




 
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:


# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests on the other host.

