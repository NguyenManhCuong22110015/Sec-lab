# Lab #2,22110015, Nguyen Manh Cuong, INSE330380E_01FIE
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

- The hashes match, confirming that the file is intact and authentic.

- Finally:

```bash
nano plaintext.txt
```

![image](https://github.com/user-attachments/assets/5eb2248e-11db-443d-9265-52384c8bc101)

 
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:

## 1. Prepare

-  Create a file named message.txt and writes the secret message "This is a secret message." into it. 

```bash
echo "This is a secret message." > message.txt 
```


## 2.  Generate RSA Key Pair on `Receiver`

- Generate a 2048-bit RSA private key. This is the private key used by the Receiver:

```bash
openssl genrsa -out private_key.pem 2048
```
- 


![image](https://github.com/user-attachments/assets/04191671-0758-403e-a68f-4f6a2fea76c4)

  
- Extract the corresponding public key:

```bash
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

![image](https://github.com/user-attachments/assets/ad6c5893-8f4f-41f5-ae96-ca5d59d84969)


## 3.The `Receiver` shares the public_key.pem file with the `Sender` using `netcat`

- In `Receiver` :

```bash
nc 172.17.0.3 1234 <  public_key.pem
```

- In `Sender` :

```bash
nc -l -p 1234 >  public_key.pem
```

![image](https://github.com/user-attachments/assets/2d8cce93-cb32-4787-96a2-5f4d244dd8fb)



## 4. Create and Encrypt Symmetric Key on `Sender`


- Create a symmetric AES key (32 bytes for AES-256):

```bash
oppenssl rand -base64 32 > symmetric_key.txt
```


![image](https://github.com/user-attachments/assets/1d24c7ec-2af9-49e5-98d0-4ac7aaf009c1)


- Encrypt the symmetric key using Computer B’s public RSA key. The Sender encrypts the symmetric key using the Receiver's public RSA key. The encrypted symmetric key is saved to encrypted_key.bin.

```bash
openssl pkeyutl -encrypt -inkey public_key.pem -pubin -in symmetric_key.txt -out encrypted_key.bin
```

- `openssl pkeyutl`: This is the command used to perform public key operations like encryption or decryption using RSA or other asymmetric algorithms.
- `encrypt`: This flag indicates that the operation is encryption. You're encrypting data with the public key.
- `inkey public_key.pem`: Specifies the input key file. Here, it's the public key file (public_key.pem) that will be used for encryption.
- `pubin`: This flag indicates that the provided key is a public key (as opposed to a private key).
- `in symmetric_key.txt`: Specifies the input file, which in this case is the symmetric key (symmetric_key.txt) that you want to encrypt.
- `out encrypted_key.bin`: Specifies the output file where the encrypted symmetric key will be saved (encrypted_key.bin).

![image](https://github.com/user-attachments/assets/b2df5fdf-52d3-4aa5-b392-3e619d009e84)


- Encrypt the file message.txt using the symmetric AES key. The Sender encrypts message.txt using AES-256-CBC with the symmetric key stored in symmetric_key.txt. The encrypted file is saved as encrypted_file.bin.

```bash
openssl enc -aes-256-cbc -salt -in message.txt -out encrypted_file.bin -pass file:symmetric_key.txt -pbkdf2
```

- `aes-256-cbc:` Specifies the encryption algorithm and mode.
- `salt`: This flag tells OpenSSL to use a salt when generating the encryption key. Salt is random data added to the input before encryption to ensure that identical plaintexts result in different ciphertexts, preventing certain types of attacks.
- `in message.txt`: Specifies the input file to be encrypted (in this case, message.txt).
- `out encrypted_file.bin`: Specifies the output file for the encrypted data (saved as encrypted_file.bin).
- `pass file:symmetric_key.txt`: The flag -pass is used to provide a passphrase for encryption. Here, it points to the file symmetric_key.txt, which contains the symmetric key used for encryption.
- `pbkdf2`: This option enables PBKDF2 (Password-Based Key Derivation Function 2), which is a cryptographic algorithm to derive keys from passwords. It adds extra security by using multiple iterations and salt to generate the ke


![image](https://github.com/user-attachments/assets/ac9458d1-d50d-4757-b4c5-c7a7b4454d29)


## 5. `The Sender` send both `encrypted_key.bin` and `encrypted_file.bin` to `the Receiver`

- In receiver. The Receiver listens on port 1234 for both the encrypted symmetric key (encrypted_key.bin) and the encrypted file (encrypted_file.bin).: 

```bash
nc -l -p 1234 > encrypted_key.bin
nc -l -p 1234 > encrypted_file.bin
```

- In sender. The Sender sends both the encrypted_key.bin and encrypted_file.bin to the Receiver over netcat to the IP address 172.17.0.2 on port 1234

```bash
nc 172.17.0.2 1234 < encrypted_key.bin
nc 172.17.0.2 1234 < encrypted_file.bin
```

- 2 files have been sent successfully
  
![image](https://github.com/user-attachments/assets/ddf3cdd0-2222-4ff8-8772-02b89c28d100)

  

## 6. Decrypt Symmetric Key on Receiver

- The Receiver uses their private RSA key to decrypt the encrypted_key.bin file, retrieving the symmetric key and saving it to symmetric_key.txt

```bash
openssl pkeyutl -decrypt -inkey private_key.pem -in encrypted_key.bin -out symmetric_key.txt
```
- `decrypt`: This flag indicates that the operation is decryption. You're decrypting the data using the private key.
- `inkey private_key.pem`: Specifies the private key file used for decryption (private_key.pem).
- `in encrypted_key.bin`: Specifies the encrypted data file (encrypted_key.bin) that you want to decrypt.
- `out symmetric_key.txt`: Specifies the output file where the decrypted symmetric key will be saved (symmetric_key.txt


![image](https://github.com/user-attachments/assets/cc8fb5c5-dc48-4f04-9620-7f038bd882bc)

![image](https://github.com/user-attachments/assets/154b6024-773f-4ec4-93ad-d5c1813418b1)


## 7. Decrypt the File on Reciver

- The Receiver decrypts the encrypted file encrypted_file.bin using the symmetric key stored in symmetric_key.txt and saves the decrypted content to decrypted_example.txt.

```bash
openssl enc -d -aes-256-cbc -in encrypted_file.bin -out decrypted_example.txt -pass file:symmetric_key.txt -pbkdf2
```

- `d`: This flag specifies that the operation is decryption (as opposed to encryption).
- `aes-256-cbc`: Specifies the algorithm (AES) and mode (CBC) used for decryption, just like in the encryption process.
- `in encrypted_file.bin`: Specifies the input file to be decrypted (encrypted_file.bin).
- `out decrypted_example.txt`: Specifies the output file where the decrypted content will be saved (decrypted_example.txt).
- `pass file:symmetric_key.txt`: Specifies the file that contains the symmetric key to be used for decryption (symmetric_key.txt)

![image](https://github.com/user-attachments/assets/a4aa1c95-0635-49c6-b79b-943d5bea44dc)


and 

```bash
nano decrypted_example.txt
```

![image](https://github.com/user-attachments/assets/6ec0a3cc-743d-40a1-84c0-1c559f790ec7)

### Conclusion

- The public key is shared securely between the Sender and the Receiver.
- The symmetric key used for file encryption is encrypted using the Receiver's public RSA key and sent to the Receiver.
- The file (message.txt) is encrypted using the symmetric key, ensuring confidentiality.
- The Receiver successfully decrypts both the symmetric key and the file, retrieving the original message "This is a secret message."



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





