Title: Cracking the Confusion: Encryption and Tokenization for Data Centers, Servers, and Applications

#The New Age of Encryption

Data encryption has long been part of the information security arsenal. From passwords, to files, to databases, we rely on encryption to protect our data in storage and on the move. It's a foundational element in any security professional's education. But despite its long history and deep value, adoption inside data centers and applications has been relatively -- even surprisingly -- low.

Today we see encryption growing in the data center at an accelerating rate, due to a confluence of reasons. A trite way to describe it is "compliance, cloud, and covert affairs". Organizations need to keep auditors off their backs; keep control over data in the cloud; and stop the flood of data breaches, state-sponsored espionage, and government snooping (even their own).

And thanks to increasing demand, there is a growing range of options, as vendors and even free and Open Source tools address the opportunity. We have never had more choice, but with choices comes complexity; and outside of your friendly local sales representative, guidance can be hard to come by.

For example, given a single application collecting an account number from each customer, you could encrypt it in any of several different places: the application, the database, or storage -- or use tokenization instead. The data is encrypted (or substituted), but each place you might encrypt raises different concerns. What threats are you protecting against? What is the performance overhead? How are keys managed? Does it meet compliance requirements?

This paper cuts through the confusion to help you pick the best encryption options for your projects. In case you couldn't guess from the title, our focus is on encrypting in the data center -- applications, servers, databases, and storage. Heck, we will even cover cloud computing (IaaS: Infrastructure as a Service), although [we covered that in depth in another paper](https://securosis.com/Research/Publication/defending-cloud-data-with-infrastructure-encryption). We will also cover tokenization and its relationship with encryption.

We won't cover encryption algorithms, cipher modes, or product comparisons. We *will* cover different high-level options and technologies, such as when to encrypt in the database vs. in the application, and what kinds of data are best suited for tokenization. We will also cover key management, some essential platform features, and how to tie it all together.

#Understanding Encryption Systems

When most security professionals first learn about encryption the focus is on keys, algorithms, and modes. We learn the difference between symmetric and asymmetric and spend a lot of time talking about Bob and Alice.

Once you start working in the real world your focus needs to change. The fundamentals are still important but now you need to put them into practice as you implement *encryption systems* -- the combination of technologies that actually protects data. Even the strongest crypto algorithm is worthless if the system around it is full of flaws.

Before we go into specific scenarios let's review the basic concepts behind building encryption systems because this becomes the basis for decisions on which encryption options to go use.

##The Three Components of a Data Encryption System

When encrypting data, especially in applications and data centers, knowing *how* and *where* to place these pieces is incredibly important, and mistakes here are one of the most common causes of failure. We use all our data at some point, and understanding where the exposure points are, where the encryption components reside, and how they tie together, all determine how much actual security you end up with.

Three major components define the overall structure of an encryption system.

* **The data:** The object or objects to encrypt. It might seem silly to break this out, but the security and complexity of the system depend on the nature of the payload, as well as where it is located or collected.
* **The encryption engine:** This component handles actual encryption (and decryption) operations.
* **The key manager:** This handles keys and passes them to the encryption engine.

In a basic encryption system all three components are likely located on the same system. As an example take personal full disk encryption (the built-in tools you might use on your home Windows PC or Mac): the encryption key, data, and engine are all stored and used on the same hardware. Lose that hardware and you lose the key and data -- and the engine, but that isn't normally relevant. (Neither is the key, usually, because it is protected with another key, or passphrase, that is *not* stored on the system -- but if the system is lost while running, with the key in memory, that becomes a problem). For data centers these major components are likely to reside on different systems, increasing complexity and security concerns over how the pieces work together.

##Building an Encryption System

In a straightforward application we normally break out the components -- such as the encryption engine in an application server, the data in a database, and key management in an external service or appliance.

Or, for a legacy application, we might instead enable *Transparent Database Encryption* (TDE) for the database, with the encryption engine and data both on the same server, but key management elsewhere.

All data encryption systems are defined by where these pieces are located -- which, even assuming everything works perfectly, define the protection level of the data. We will go into the different layers of data encryption in the next section, but at a high level they are:

* In the application where you collect the data.
* In the database that holds the data.
* In the files where the data is stored.
* On the storage volume (typically a hard drive, tape, or virtual storage) where the files reside.

All data flows through that stack (sometimes skipping applications and databases for unstructured data). Encrypt at the top and the data is protected all the way down, but this adds complexity to the system and isn't always possible. Once we start digging into the specifics of different encryption options you will see that defining your requirements almost always naturally leads you to select a particular layer, which then determines where to place the components.

###The Three Laws of Data Encryption

Years ago we developed the *Three Laws of Data Encryption* as a tool to help guide the encryption decisions listed above. When push comes to shove, there are only three reasons to encrypt data:

1. If the data moves, physically or virtually.
2. To enforce separation of duties beyond what is possible with access controls. Usually this only means protecting against administrators because access controls can stop everyone else.
3. Because someone says you have to. We call this "mandated encryption".

Here is an example of how to use the rules. Let's say someone tells you to "encrypt all the credit card numbers" in a particular application. Let's further say the reason is to prevent loss of data if a database administrator account is compromised, which eliminates our third reason.

The data isn't necessarily moving, but we want separation of duties to protect the database even if someone steals administrator credentials. Encrypting at the storage volume layer wouldn't help because a compromised administrative account still has access within the database. Encrypting the database files alone wouldn't help either.

Encrypting within the database is an option, depending on where the keys are stored (they must be outside the database) and some other details we will get to later. Encrypting in the application definitely helps, since that's completely outside the database. But in either cases you still need to know when and where an administrator could potentially access decrypted data.

That's how it all ties together. Know why you are encrypting, then where you can potentially encrypt, then how to position the encryption components to achieve your security objectives.

##Tokenization and Data Masking

Two alternatives to encryption are sometimes offered in commercial encryption tools: tokenization and data masking. We will spend more time on them later, but simply define them for now:

* *Tokenization* replaces a sensitive piece of data with a random piece of data that can fit the same format (such as by *looking* like a credit card number without actually being a valid credit card number). The sensitive data and token are then stored together in a highly secure database for retrieval under limited conditions.
* *Data masking* replaces sensitive data with random data, but the two aren't stored together for later retrieval. Masking can be a one-way operation, such as generating a test database, or a repeatable operation such as dynamically masking a specific field for an application user based on permissions.

For more information on tokenization vs. encryption you can [read our paper](https://securosis.com/Research/Publication/tokenization-vs.-encryption-options-for-compliance).

That covers the basics of encryption systems. Our next section will go into details of the encryption layers above before delving into key management, platform features, use cases, and the decision tree to pick the right option.

#Encryption Layers

Picture your enterprise applications as a layer cake; applications sit atop databases, databases atop files, and files mapped onto storage volumes. You can use encryption at each of these layers in your application stack; within the application, in the database, for files or on the storage volumes. *Where* you use the encryption engine in your environment is the dominant factor for security and performance. The higher up the stack offers potentially more security, but at greater complexity and performance impact.

There is a similar tradeoff made with the encryption engine and key manager deployments; more tightly coupled systems offer less complexity, but less security and reliability. Building an encryption system requires a balancing act between security, complexity and performance. Let's take a closer look at each layer, and the tradeoffs to consider:

###Application Encryption

One of the more secure ways to encrypt application data is to collect it in the application, send it to an encryption server or appliance (or embed encryption library in the application code), and then store the encrypted data in a separate database. In this way the application has full control over who sees what, and the data remains secure regardless of the security of the underlying database, file system or storage volumes. The keys themselves might be on the encryption server or could even be stored in yet another system. The separate key store increases security, simplifies management of multiple encryption appliances, and helps keep keys safe for data movement -- backup, restore, and migration/synchronization to other data centers.

###Database Encryption

Relational database management systems (RDBMS) typically have two options for encryption: transparent and columnar. In our layer cake model above, columnar encryption occurs as applications inserts data into a database, whereas transparent occurs as the database writes data out. Transparent encryption is applied automatically to data before it is stored at the file or disk layer. In this model encryption and key management happens behind the scenes without the user's knowledge or explicit application programming needed. The database management system handles the encryption and decryption operations as data is read (or written), ensuring _all_ data is secured, and offering very good performance. When you need finer control over data access, you can encrypt single columns -- or tables -- within the database. This approach offers an advantage that only authenticated users of encrypted data will be able to gain access, but at a cost of changing database or application code to manage encryption operations. With either approach, there is less burden on application developers to build a crypto system, but slightly less control over who can access sensitive data.

Some third-party encryption tools also offer transparent database encryption by automatically encrypting the data as it is stored in files. These tools aren't part of the database management system itself, so they can work with databases that don't support TDE, and provide greater separation of duties from database administrators.

###File Encryption

Some applications, such as payment systems and web applications, do not use databases, and store sensitive data into files. Encryption is applied transparently as data is saved into files. This type of encryption is offered as a third party add-on to the file system, or embedded within the operating system. Encryption and decryption are transparent to the user and the application. Data is decrypted when a user requests a file, and after they have authenticated to the system. If the user does not have permission to read the file, or has not provided proper credentials, they only get encrypted data. File encryption is commonly used to protect 'data at rest' in applications that do not include encryption capabilities, including legacy enterprise applications and many big data platforms.

##Disk/Volume Encryption

Many off-the shelf disk drives and storage-area-network (SAN) arrays include automatic data encryption. Encryption is applied as data is written to disk, and decrypted by authenticated users/apps when requested. Most enterprise class systems hold the encryption key locally to support encryption operations, but rely upon external key management services to manage keys and provide advanced key services, such as key rotation. Volume encryption protects data in the event that the drives are physically stolen. Authenticated users and applications will be provided unencrypted copies of files and data.


##Tradeoffs

In general, the further 'up the stack' you deploy encryption, the more secure your data is. The price of that extra security is more difficult integration efforts, usually in application code changes. Ideally we would encrypt all data at the application layer, and fully leverage user authentication, authorization and business context to determine who gets to see sensitive data. In the real-world, the code changes required for this level of precision are often insurmountable from an engineering perspective, not to mention cost prohibitive. And what is often counter-intuitive, transparent encryption variants perform _faster_, even with larger data sets, than application layer encryption. The tradeoff is moving high enough 'up the stack' to address relevant threats, while minimizing the pain of integration and management. We'll walk you through the selection process in detail later in this series.

Next up in our series: A look at key management options.
