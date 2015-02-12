
#Encryption Layers 	

Picture your enterprise applications as a layer cake; applications sit atop databases, databases atop files, and files mapped onto storage volumes. You can use encryption at each of these layers in your application stack; within the application, in the database, for files or on the storage volumes. *Where* you use the encryption engine in your environment is the dominant factor for security and performance. The higher up the stack offers potentially more security, but at greater complexity and performance impact. 

There is a similar tradeoff made with the encryption engine and key manager deployments; more tightly coupled systems offer less complexity, but less security and reliability. Building an encryption system requires a balancing act between security, complexity and performance. Let's take a closer look at each layer, and the tradeoffs to consider: 
 	
##Application Encryption

One of the more secure ways to encrypt application data is to collect it in the application, send it to an encryption server or appliance (or embed encryption library in the application code), and then store the encrypted data in a separate database. In this way the application has full control over who sees what, and the data remains secure regardless of the security of the underlying database, file system or storage volumes. The keys themselves might be on the encryption server or could even be stored in yet another system. The separate key store increases security, simplifies management of multiple encryption appliances, and helps keep keys safe for data movement – backup, restore, and migration/synchronization to other data centers.

##Database Encryption

Relational database management systems (RDBMS) typically have two options for encryption: transparent and columnar. In our layer cake model above, columnar encryption occurs as applications inserts data into a database, whereas transparent occurs as the database writes data out. Transparent encryption is applied automatically to data before it is stored at the file or disk layer. In this model encryption and key management happens behind the scenes without the user’s knowledge or explicit application programming needed. The database management system handles the encryption and decryption operations as data is read (or written), ensuring _all_ data is secured, and offering very good performance. When you need finer control over data access, you can encrypt single columns - or tables - within the database. This approach offers an advantage that only authenticated users of encrypted data will be able to gain access, but at a cost of changing database or application code to manage encryption operations. With either approach, there is less burden on application developers to build a crypto system, but slightly less control over who can access sensitive data. 


##File  

Some applications, such as payment systems and web applications, do not use databases, and store sensitive data into files. Encryption is  applied transparently as data is saved into files. This type of encryption is offered as a third party add-on to the file system, or embedded within the operating system. Encryption and decryption are transparent to the user and the application. Data is decrypted when a user requests a file, and after they have authenticated to the system. If the user does not have permission to read the file, or has not provided proper credentials, they only get encrypted data. File encryption is commonly used to protect 'data at rest' in applications that do not include encryption capabilities, including legacy enterprise applications and many big data platforms. 

##Disk/Volume

Many off-the shelf disk drives and storage-area-network (SAN) arrays include automatic data encryption. Encryption is applied as data is written to disk, and decrypted by authenticated users/apps when requested. Most enterprise class systems hold the encryption key locally to support encryption operations, but rely upon external key management services to manage keys and provide advanced key services, such as key rotation. Volume encryption protects data in the event that the drives are physically stolen. Authenticated users and applications will be provided unencrypted copies of files and data. 

		
#Tradeoffs 

In general, the further 'up the stack' you deploy encryption, the more secure your data is. The price of that extra security is more difficult integration efforts, usually in application code changes. Ideally we would encrypt all data at the application layer, and fully leverage user authentication, authorization and business context to determine who gets to see sensitive data. In the real-world, the code changes required for this level of precision are often insurmountable from an engineering perspective, not to mention cost prohibitive. And what is often counter-intuitive, transparent encryption variants perform _faster_, even with larger data sets, than application layer encryption. The tradeoff is moving high enough 'up the stack' to address relevant threats, while minimizing the pain of integration and management. We'll walk you through the selection process in detail later in this series. 

Next up in our series: A look at key management options.