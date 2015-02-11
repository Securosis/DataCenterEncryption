Title: Cracking the Confusion: Encryption and Tokenization for Data Centers, Servers, and Applications

#The New Age of Encryption

Data encryption is long a part of the information security arsenal. From passwords, to files, to databases we rely on encryption to protect our data in storage and on the move. It's a foundational element of any security professional's education. But, despite the long history and deep value of data encryption, adoption inside our data centers and applications has been relatively, even surprisingly, low. 

Today we see encryption growing in the data center at an accelerating rate thanks to a confluence of reasons. The trite way to word it is, "compliance, cloud, and covert affairs". Organizations need to keep auditors off their backs, keep control over data in the cloud, and stop the flood of data breaches, state-sponsored espionage, or even their own government snooping. 

And thanks to increasing demand, there's a growing market of options as vendors and even free and Open Source tools look to meet the opportunity. There have never been more choices, but with choices comes complexity, and outside of your friendly local sales representative, guidance can be hard to come by. 

For example, given a single application collecting an account number from your customers, you could potentially encrypt it in different places; the application, in the database, in storage, or use tokenization instead. The data is encrypted, but where you encrypt presents different concerns. What threats is it protecting against? What is the performance overhead? How are keys managed? Does it meet compliance requirements? 

This paper cuts through the confusion to help you pick the best encryption options for your projects. If you couldn't guess from the title, our focus is on encrypting in the data center. Your applications, servers, databases, and storage. Heck, we'll even cover cloud computing (infrastructure), although [we covered that in depth in this paper](https://securosis.com/Research/Publication/defending-cloud-data-with-infrastructure-encryption). We'll also cover the role of tokenization, and it's relationship with encryption.

We aren't going to cover encryption algorithms, cipher modes, or product comparisons. What we *do* cover are the different high level options and technologies, like when to encrypt in the database vs. your application, or what kinds of data are best suited for tokenization. We also cover key management, some essential platform features, and how to tie it all together.

#Understanding Encryption Systems

When most security professionals first learn about encryption, the focus is on keys, algorithms, and modes. We learn the difference between symmetric and asymmetric, and spend a lot of time talking about Bob and Alice.

Once you start working in the real world, the focus needs to change. All the fundamentals are still important, but now you need to put it into practice as you implement *encryption systems* — the combination of technologies that actually protects the data. Even the strongest crypto algorithm is worthless if the system around it is full of flaws.

Before we go into specific scenarios, let's review the basic concepts behind building encryption systems since this becomes the basis for making decisions on exactly which encryption options to go with.

##The Three Components of a Data Encryption System

When encrypting data, especially in applications and data centers, knowing how and where to place these pieces is incredibly important, and one of the most common causes of failure. We use all of our data at some point, and understanding where the exposure points are, where the encryption components reside, and how they tie together all determine how much actual security you end up with.

Three major components define the overall structure of an encryption system are:

* **The data:** The object or objects to encrypt. It might seem silly to break this out, but the security and complexity of the system are influenced by the nature of the payload, as well as where it is located or collected.
* **The encryption engine:** The component that handles the actual encryption (and decryption) operations.
* **The key manager:** The component that handles key and passes them to the encryption engine.

In a basic encryption system all three components are likely to be located on the same system. As an example take personal full disk encryption (the built-in tools you might use on your home Windows PC or Mac): the encryption key, data, and engine are all stored and used on the same hardware. Lose that hardware and you lose the key and data -- and the engine, but that isn't normally relevant. (Neither is the key, usually, because it is protected with another key, or passphrase, that is *not* stored on the system -- but if the system is lost while running, with the key is in memory, that becomes a problem). For data centers, it's likely these major components will reside on different systems, increasing complexity and security concerns over how the three pieces work together. 

##Building an Encryption System

In a straightforward application we normally break out the components -- such as the encryption engine in an application server, the data in a database, and key management in an external service or appliance.

Or, for a legacy application, we might instead enable *Transparent Database Encryption* (TDE) for the database, where the encryption engine and data are both on the same server, but the key management is external. 

All data encryption systems are defined by where these pieces are located, which, even assuming everything works perfectly, define the protection level of the data. We will go into the different layers of data encryption in the next section, but at a high level they are:

* In the application where you collect the data.
* In the database that holds the data.
* In the files where the data is stored.
* On the storage volume (e.g. hard drive, tape, or virtual storage) where the files reside.

All data flows through that stack (sometimes skipping applications and databases for unstructured data). Encrypt at the top and the data is protected all the way down, but this adds complexity to the system and isn't always possible. Once we start digging into the specifics of different encryption options, you'll see that picking the defining your requirements almost always naturally leads you to selecting a particular layer, which then defines where to place the components.

###The Three Laws of Data Encryption

Years ago we developed the *Three Laws of Data Encryption* as a tool to help guide the encryption decisions we list above. When push comes to shove, there are only three reasons to encrypt data:

1. If the data moves, physically or virtually.
2. To enforce separation of duties beyond what's possible with access controls. Usually this only means protecting against administrators, since access controls can stop everyone else. 
3. Because someone tells you you have to. We call this "mandated encryption".

Here's an example of how to use it. Let's say someone tells you to "encrypt all the credit card numbers" in a particular application. Let's further say the reason is to prevent loss of data if a database administrator account is compromised, which eliminates our third reason.

The data isn't necessarily moving, but we want to gain separation of duties to protect the database even if someone steals administrator credentials. Encrypting at the storage volume layer won't help, since a compromised admin account still has access within the database. Encrypting the database files alone won't help either.

Encrypting within the database is an option, depending on where the keys are stored (must be outside the database) and some of the other details we will get to a little later. Encrypting in the application definitely helps, since that's completely outside the database. Although in both cases, you still need to know when and where an admin could still potentially access decrypted data.

That's how it all ties together. Know why you are encrypting, then where you can potentially encrypt, then how to position the encryption components to hit your security objectives. 

##Tokenization and Data Masking

There are also two alternatives to encryption that are sometimes part of commercial encryption tools -- tokenization, and data masking. We will spend more time on them later, but define them now:

* *Tokenization* replaces a sensitive piece of data with a random piece of data that can fit the same format (e.g. *look* like a credit card number, but not actually be a credit card number). The sensitive data and the token are then stored together in a highly secure database for retrieval under limited conditions.
* *Data masking* replaces sensitive data with random data, but the two aren't stored together for later retrieval. it can be a one-way operation, such as generating a test database, or a repeatable operation, such as dynamically masking a specific field for an application user based on their permissions.

For more information on tokenization vs. encryption you can [read our paper here](https://securosis.com/Research/Publication/tokenization-vs.-encryption-options-for-compliance).

And that covers the basics of encryption systems. In our next section we will go into the details of the encryption layers we covered briefly above, before delving into key management, platform features, use cases, and the decision tree to pick the (hopefully) best option.

