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

Picture enterprise applications as a layer cake: applications sit on databases, databases on files, and files are mapped onto storage volumes. You can use encryption at each of these layers in your application stack: within the application, in the database, on files, or on storage volumes. *Where* you use an encryption engine dominates security and performance. Higher up the stack can offer more security, with higher complexity and performance cost.

There is a similar tradeoff with encryption engine and key manager deployments: more tightly coupled systems offer less complexity, but less security and reliability. Building an encryption system requires a balance between security, complexity, and performance. Let's take a closer look at each layer and their tradeoffs.

###Application Encryption

One of the more secure ways to encrypt application data is to collect it in the application, send it to an encryption server or appliance (an encryption library embedded in the application), and then store the encrypted data in a separate database. The application has full control over who sees what and can secure data without depending on the security of the underlying database, file system, or storage volumes. The keys themselves might be on the encryption server or could even be stored in yet another system. The separate key store increases security, simplifies management of multiple encryption appliances, and helps keep keys safe for data movement -- backup, restore, and migration/synchronization to other data centers.

###Database Encryption

Relational database management systems (RDBMS) typically have two encryption options: transparent and column. In our layer cake above columnar encryption occurs as applications insert data into a database, whereas transparent encryption occurs as the database writes data out. Transparent encryption is applied automatically to data before it is stored at the file or disk layer. In this model encryption and key management happen behind the scenes, without the user's knowledge or requiring application programming. The database management system handles encryption and decryption operations as data is read (or written), ensuring _all_ data is secured, and offering very good performance. When you need finer control over data access, you can encrypt single columns, or tables, within the database. This approach offers the advantage that only authenticated users of encrypted data are able to gain access, but requires changing database or application code to manage encryption operations. With either approach there is less burden on application developers to build a crypto system, but slightly less control over who can access sensitive data.

Some third-party tools also offer transparent database encryption by automatically encrypting data as it is stored in files. These tools aren't part of the database management system itself, so they can work with databases that don't support TDE directly, and provide greater separation of duties for database administrators.

###File Encryption

Some applications, such as payment systems and web applications, do not use databases and instead store sensitive data in files. Encryption is applied transparently as data is written to files. This type of encryption is offered as a third-party add-on to the file system, or embedded within the operating system. Encryption and decryption are transparent to both users and applications. Data is decrypted when a user requests a file, after they have authenticated to the system. If the user does not have permission to read the file, or has not provided proper credentials, they only get encrypted data. File encryption is commonly used to protect "data at rest" in applications that do not include encryption capabilities, including legacy enterprise applications and many big data platforms.

##Disk/Volume Encryption

Many off-the-shelf disk drives and Storage Area Network (SAN) arrays include automatic data encryption. Encryption is applied as data is written to disk, and decrypted by authenticated users/apps when requested. Most enterprise-class systems hold encryption keys locally to support encryption operations, but rely on external key management services to manage keys and provide advanced key services such as key rotation. Volume encryption protects data in case drives are physically stolen. Authenticated users and applications are provided unencrypted copies of files and data.


##Tradeoffs

In general, the further "up the stack" you deploy encryption, the more secure your data is. The price of that extra security is more difficult integration, usually in the form o application code changes. Ideally we would encrypt all data at the application layer and fully leverage user authentication, authorization, and business context to determine who can see sensitive data. In the real world the code changes required for this level of precision control are often insurmountable engineering challenges and/or cost prohibitive. Surprisingly, transparent encryption often perform _faster_ than application-layer encryption, even with larger data sets. The tradeoff is moving high enough "up the stack" to address relevant threats while minimizing the pain of integration and management. Later in this series we will walk you through the selection process in detail.

#Key Management Options

As we mentioned back in the opening, the key (pun intended, forgive us) to an effective and secure encryption system is proper placement of the components. Of those, the one that most defines the overall system is the key manager.

Technically you can encrypt without a dedicated key manager. We know of numerous applications that take this approach. We also know of numerous applications that break, fail, or get breached. You will nearly always want to use a dedicate key management option, which breaks down into four categories:

The first thing to consider is how you want deploy external key management. There are four options:

* An HSM or other hardware key management appliance. This provides the highest level of physical security. It is the most common option in sensitive scenarios, such as financial services and payments. The HSM or appliance runs in your data center, and you always want more than one for backup. Lose access, and you lose your keys. Apple, for example, has stated publicly that they physically destroy the admin access smart cards after configuring a new appliance so no one can ever access and compromise the keys, which are destroyed if someone tries to open the housing or certain other access methods. A *hardware root of trust* is the most secure option, and all products also include hardware acceleration for cryptographic operations to improve performance.
* A key management virtual appliance. Your vendor provides a pre-configured virtual appliance (instance) for you to run where you need it. This reduces costs and increases deployment flexibility, but isn't as secure as dedicated hardware.  If you decide to go this route, use a vendor that takes exceptional memory protection precautions since there are known techniques to pull a key from memory in certain virtualization scenarios. A virtual appliance doesn't offer the same physical security as a physical server, but they do come hardened and support more flexible deployment options -- you can run it within a cloud or virtual datacenter. Some systems also allow you to use an appliance as the hardware root of trust for your keys, but then distribute keys to virtual appliances to improve performance in distributed scenarios (for virtualization, or mere cost savings).
* Key management software, which can run either on a dedicated server or within a virtual/cloud server. The difference between software and a virtual appliance is that you install the software yourself rather than receiving a configured and hardened image. Otherwise it offers the same risks and benefits as a virtual appliance, assuming you harden the server as well as the virtual appliance.
* Key management Software as a Service (SaaS). Multiple vendors now offer key management as a service specifically to support public cloud encryption. This also works for other kinds of encryption, including private clouds, but most usage is for public clouds. 

##Client access options

Whatever deployment model you choose, you need some way of getting the keys where they need to be, when they need to be there, for cryptographic operations. 

Clients (whatever needs the key) usually need support for the following core functions for a complete key management lifecycle:

* Key generation
* Key exchange (gaining access to the key)
* Additional key lifecycle functions, such as expiring or rotating a key

Depending on what you are doing, you will allow or disallow these functions under different circumstances. For example you might allow key exchange for a particular application, but not allow it any other management functions (such as generation and rotation).

Access is managed one of three ways, and many tools support more than one:

* **Software agent:** A dedicated agent handles the client's side of the key functions. These are generally designed for specific use cases -- such as supporting native full disk encryption, specific backup software, various database platforms, and so on. Some agents may also perform cryptographic functions to additional hardening such as wiping the key from memory after each use.
* **Application Programming Interfaces:** Many key managers are used to handle keys from custom applications. An API allows you to access key functions directly from application code. Keep in mind that APIs are not all created equal -- they vary widely in platform support, programming languages supported, the simplicity or complexity of the API calls, and the functions accessible via the API.
* **Protocol & standards support:** The key manager may support a combination of proprietary and open protocols. Various encryption tools support their own protocols for key management, and like a software agent, the key manager may include support -- even if it is from a different vendor. Open protocols and standards are also emerging but not in wide use yet, and may be supported.

We've written a lot about key management in the past. To dig in deeper, take a look at [Pragmatic Key Management for Data Encryption](https://securosis.com/research/publication/pragmatic-key-management-for-data-encryption) and [Understanding and Selecting a Key Management Solution](https://securosis.com/research/publication/understanding-and-selecting-a-key-management-solution).

#Platform Features and Options

The encryption engine and the key store are the major functional pieces in any encryption system, but there are supporting systems with any data center encryption solution that are important for both overall management, as well as tailoring the solution to fit within your application infrastructure. Here we discuss the major supporting components and variations. 

##Central Management

For enterprise class data center encryption, you need a central point to define both what data is to be secured, as well as defined key management policies. As such, management tools provide a window onto what data is encrypted, and setting usage policies for cryptographic keys. You can think of this as governance of the entire crypto eco-system, including key rotation policies, integration with identity management, and what IT admins are authorized to do. Some products even provide the ability to manage remote cryptographic engines and automatically apply encryption as data is discovered. Management interfaces have evolved to accommodate both security and IT management to set policy without needing to be an expert with cryptographic subsystems. The larger and more complex your environment, the more critical central management becomes in order to get control of your environment without making it a full time job. 

##Format Preserving Encryption

Encryption protects data by scrambling it into an unreadable state. Format Preserving Encryption (FPE) also scrambles data into an unreadable state, but allows you to keep the _format_ of the original data. For example, if you are using FPE to encrypt a 9 digit social security number, the encrypted result would be 9 characters long as well. And, as all of the commercially available FPE tools are variations of AES encryption, it remains nearly impossible to break, and the original data cannot be recovered without the key.  The principle reason to use FPE is to avoid re-coding applications and re-structuring databases to accommodate encrypted (binary) data. Both tokenization and FPE offer this advantage. But encryption obfuscates sensitive information, while tokenization removes it entirely (to another location). Should you need to propagate copies of sensitive data yet still control occasional access, FPE is a good option. Keep in mind, FPE is still encryption, and sensitive data is still present.
		
##Tokenization

Tokenization is a method of replacing sensitive data with non-sensitive placeholders: tokens. Tokens are created to look exactly like the value they replace, retaining both format and data type. Tokens are commonly ‘random’ values that take the form of the original data but lack intrinsic value. For example, a token that looks like a credit card number cannot be used as a credit card to process financial transactions. Its only value is as a reference to the original value stored in the token server that created and issued the token. These tokens are usually swapped in for sensitive data stored in relational databases and files, allowing applications to continue to function without changes, and simultaneously remove the risk of a data breach. Tokens may even include elements of the original value to facilitate processing. Tokens may also be created from 'code-books' or one-time-pads; the tokens are still random, but maintain a mathematical relationship to the original, blurring the line between random numbers and FPE. Tokenization has become a very popular, and effective, means of reducing exposure of sensitive data. 


##Masking 

Masking, like tokenization, replaces sensitive data with similar non-sensitive values. And like tokenization, masking provides data that looks and acts like the original data, but in it's randomized state doesn’t pose a risk of exposure. But masking solutions are not goes one step further, protecting sensitive data _elements_ while maintaining the value of the aggregate data _set_. For example, we may  replace real user names in a file with those randomly selected from a phone directory, skew a persons date of birth by some number of days, or shuffle employee salaries in a database column so they are associated with random employees. This means reports and analytics will continue to run, and produce _meaningful results_, yet database as a whole is protected. Masking platforms commonly take a copy of production data, mask it, and then move the copy to another server. This is called static masking, or sometimes described by the process of 'extraction-transformation-load', or ETL for short. 

A recent variation is called 'dynamic masking'; this is where masks are applied as data is read from a database or file. With dynamic masking the original files or databases remain untouched, but the results are changed. For example, depending upon the requestors credentials, their request may return the original - real - values, or the masked copy. In this later case, data is dynamically substituted for a non-sensitive surrogate. Most dynamic masking platforms are a 'proxy' that acts as a kind of firewall, using redaction to ensure a speedy return of information. In select systems, you may find more intelligent randomization, or tokenization, or even FPE. 

Again, the lines between FPE, tokenization and masking are blurring as new technical variants emerge. But it should be clear that tokenization and masking variants offer superior value for cases where you don't want sensitive data exposed, yet cannot risk changes to your applications. 

#Top Encryption Use Cases

Encryption, like most security, is only adopted after there is a business need. It may be a need to keep corporate data secret, protect customer privacy, ensure data integrity, or because there is a compliance mandate that requires you to protect data; but there is always a motivating factor that drives companies to adopt encryption. The principle use cases have changed over the years, but the following list are commonly cited reasons that prompt customer to adoption encryption. 

##Databases

Protecting data stored within databases is a top use case, be it mainframe, relational or NoSQL databases. The motivation may be to combat data breaches, keep administrators honest, support multi-tenancy, meet contractual obligations or even comply with state mandated privacy laws. Surprisingly, database encryption is a relatively new phenomena. Database administrators historically viewed encryption as unacceptable performance overhead, and data security professionals view database encryption as a redundant control, only effective in the event firewalls, identity management and other security measures failed. The steady stream of data breaches only recently shattered this false impression. Combined with continued performance advancements, multiple deployment options, and general platform maturity, database encryption no longer carries a stigma. Today, with data sprawl across hundreds of internal databases, test systems and third party service providers, organization use a mixture of encryption, tokenization and data masking to provide tailored protection for each potential threat regardless of where a database is moved/used. 

##Cloud storage

Despite corporate reservations about compliance and security of multi-tenant cloud services, employees looking to take advantage of the low cost and incredible convenience of cloud storage, have adopted solutions without IT's knowledge. File sharing services like Dropbox, data storage services like Amazon's S3, and productivity tools like Google Docs became widespread seemingly overnight. Data encryption is the principle tool companies employ to secure the data that moves into the cloud. They leverage both on-premise and cloud-native capabilities, bundled with identity and key management services, both compartmentalize employee access, as well as limit access from 3rd party cloud admins. Requests for access are logged, with data storage and decryption events shared with security and compliance tools. 

##Compliance 

Compliance is a principle driver for sales of encryption and tokenization. Regulations like Sarbanes-Oxley (SOX) and contractual obligations like the Payment Card Industry Data Security Standard (PCI-DSS) remain the principle reasons companies invest. Typical policies cover IT administrators accessing data files, users issuing ad hoc queries, retrieval of "too much" information, or any examination of restricted data elements such as credit card numbers. As such, compliance controls typically focus on issues of privileged user entitlements (what users can access), segregation of duties (admins can't read data), and the security of data as it is moved between application and database instances. These policies are commonly enforced by the applications which process users requests, limiting access (decryption) based upon a documented policy. The policy's may be as simple as which users are allowed to see certain types of data, while others build-in fraud deterrence, limiting how many records a user is allowed to see, or perhaps shutting off access entirely if a users behavior is deemed suspicious. In other use cases, where companies move sensitive data to 3rd party systems they do not control, use of data masking and tokenization have become popular choices, ensuring sensitive data does not leave the company at all. 

##Payments 

Placeholder

##Applications

Every company is reliant on applications to one degree or another, and these applications process data that is critical to the business. Most applications, be it 'Web' or 'Enterprise', leverage encryption. Encryption capabilities may be embedded in the application, or bundled with the underlying file system, storage array, or relational database system. Application encryption is selected in those cases where fine grained control is needed, specifically encrypting select elements of data, and only decrypting that information when it makes sense with the business context of the application - and not because any old user provided credentials. The granularity of control comes at a price, and that means it is more difficult to implement, and changes in usage policies mean code changes to the application, requiring extensive validation testing. While the operational costs may be steep, this level of security is essential for some applications, especially those used in financial or payment applications. For other types of applications, simply protecting data 'at rest' (_i.e._: in files or databases), with transparent encryption at the file or database layer, is suitable.