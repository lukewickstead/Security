# OWASP Top 10 #7 - Insecure Cryptographic Storage #

## What Is It? ##

Data which is sensitive should be stored cryptographically secure.

Not only should sensitive data be encrypted or hashed, but it should also be hashed or encrypted with an algorithm which requires a reasonable level of processing to decipher.

## Hashing ##

### What Is Hashing? ###

Hash functions take some input data and generates a fixed-length hash based upon it.

Hashing is a one way process; you can never reveal the original data by reversing the process. 

Hashing is used to store data which is not required to be revealed such as passwords. Passwords should be stored within the database hashed. Comparison to them should be made by comparing a hashed version of the password provided during login.

Hashing can be used with a salt. A salt is random seed to be used during the hashing process. This allows the predictability of the hashed result to appear random. The salt will need to be stored along with the password.

### How Is It Exploited ###

Firstly and unbelievably we are still seeing websites having sensitive data such as passwords leaked where the contents are not hashed. Details such as passwords should never be stored as plain text.
 
When hashed passwords and their salts are leaked or gained by hackers, their contents are deciphered by simply trying to hash the contents of  known passwords, words and phrases and comparing the results to the leaked hashed passwords. Variations of these passwords, words and phrases are made by replacing some letters with numbers and punctuation characters; for example L with ! and O (letter o) with 0 (zero).

This is a brute force approach to cracking passwords but as hashing is a one way process, it is the only way.

### How Can I Protect Myself From It? ###

#### Always Hash Passwords ####
 
All passwords should be hashed. Under no circumstances should passwords be stored as plain text.

#### Select A Suitably Strong Hashing Algorithm ####

There is no such thing as a safe password. Given the right amount of processing power and time any password can be cracked.  

A hashing algorithm which is securer than another simply means it takes longer to perform the hash. By increasing the time it takes to perform a hash, the less attempts a hacker can make in a certain period. Therefore reducing the passwords that can be cracked before the breach is realised and the site is taken offline.
 
MD5 and SHA-1 are common hashing algorithms used today but these algorithms are considered weak and should be replaced by options within the AES (Advanced Encryption Standard) such as SHA-256. 

MVC 4 uses PBKDF2 which allows applying the hashing algorithm up to 1000 times. This means that the hashing process is performed up to 1000 times upon the data before being considered the final hashed entity. This means that cracking passwords is up to 1000 times slower.

The BCrypt algorithm allows defining the amount of CPU processing power which is required to hash a password. 

Increase the CPU processing power required to hash a password does mean end user access is slower during login but it also means that bulk cracking of passwords is a lot slower. 

#### Never Write Your Own ####

Never write your own hashing algorithms, use tried and trusted industry algorithms from tried and trusted third party frameworks. 

Keep third party frameworks which are used to provide hashing algorithms up to date.

For implementing a login process within .NET use the .NET authorisation system as this uses industry best practices.

#### Strong Passwords ####

A hacker's only means of cracking passwords is to use a known list of passwords, words and phrases along with variations made by replacing some letters with numbers and symbols.

The more complex and unique a user's password is, the less likely hood they can be cracked even if a hashed version of them has been leaked.

## Encryption ##

Encryption is used when the contents of the data being encrypted should be hidden from unwanted parties but should also be unencrypted to be viewed by required parties.

There are two types of encryption; symmetric and asymmetric.

Hashing is not encrypting as you can not undo the hash process to reveal the original data.

### Symmetric And Asymmetric Encryption ###

Symmetric involves using a single private key to both encrypt and decrypt the data. This form of encryption is normally used internally where keys are not required to be distributed

Asymmetric has both a public and private key. This is often referred to as public key encryption. A good example of a process which uses this is SSL/HTTPS.

key management is very important to ensure that keys are kept safe. However if a website is breached then the private key would probably be taken allowing the encryped data to be easily unencrypted. Hashing does not have this problem as there are no keys and no unencrypting. A hacker would need to crack the passwords. Therefore encryption should not be used to store passwords. 

### Windows Data Protection API ###

The windows Data protection API uses the machine as the private key so there is no generation of a private key and as such it cannot be leaked.

https://msdn.microsoft.com/en-gb/library/ms995355.aspx

Data can be encrypted with the Protect method:

```

var unencryptedBytes = Encoding.Unitcode.GetBytes(unencryptedString);
var encryptedBytes = ProtectedData.Protect(unencryptedBytes, null, DataProtectionScope.LocalMachine);
```

Encrypted data can be unencrypted with the Unprotect method:

```

var unencryptedBytes = ProtectedData.Unprotect(encryptedBytes, null, DataProtectionScope.LocalMachine);
var unencryptedString = Encoding.Unicode.GetString(unencryptedBytes)
```


The DataProtectionScope enumerator value can be set to LocalMachine or CurrentUser. If data should only be unencrypted by the current user then the later should be used.