# Password Handling

## Learning Goals

- Explain the differences between encoding, encrypting, and hashing
- Introduce password hashing in PostgreSQL.

## Introduction

Before we continue writing code in our Spring Security project, we should
address an important topic in security.

Let's talk about passwords.

Passwords are considered sensitive information as it could help a hacker login
to an account. For example, if the password used for an online bank account got
out, that would be bad news. When creating applications that handle passwords
and storing these passwords, we need to be careful with how this information is
passed around.

How can we protect our passwords and other sensitive data in a more secure way?
Let's first define some common words when it comes to securing passwords.

## Encoding, Encrypting, and Hashing

Encoding and encrypting and hashing - oh my!

These security buzzwords are usually confused to mean the same thing. But these
actually are different techniques. Let's look at the definitions of each:

**Encoding**: This technique transforms data from one format to another so that
it can be consumed by other systems. This technique is not used for security
purposes and can be easily reversed. An example of encoding is if we look at an
ASCII table:

![ASCII Table](https://curriculum-content.s3.amazonaws.com/java-mod-2/chars-and-strings/ASCII-Table.png)

[Medium: ASCII Table in Java Programming](https://medium.com/@aidafarihabaharunsuratman/did-someone-actually-use-ascii-table-in-java-programming-9710a65c6ed9)

We can easily translate the uppercase character `A` to the decimal value of 65
and back.

**Encrypting**: This technique makes the data unreadable and difficult to decode
to an unauthorized reader. It makes use of a secret key in order to transform
the data back to its original state.

A simple example of encryption: Say Suzie wants to send a secret message to
Dustin. She doesn't want to encode it in case something evil tries to intercept
it. In order to do so, she creates a key to go along with the message. Dustin
also knows the key, so he can decrypt the message to read it. If it was
intercepted, the message would look like gibberish to the unauthorized
recipient as long as they also don't have the key.

So while encryption is used for security purposes, it can also be reversed if
the key is known.

**Hashing**: This technique takes the data and generates a `String` that is
specifically based on the input of the data. If the input is changed, even just
slightly, the hash will also change. This ensures data integrity. We can
generate this hash by using a hashing algorithm. Once the input is put through
the hashing algorithm, the data _cannot_ be reversed back. This makes this
technique one of the most secure ways to handle passwords!

## Password Hashing in PostgreSQL

Let's create a database that we can connect to in order to simulate a user
logging into an application! Open up pgAdmin4 and create a new database. Let's
call it "security_demo".

![create-security-demo-db](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/create-sercurity-demo-db.png)

Once the database has been created, open up the Query Tool and copy in the
following to create the `users` table and a user with the same credentials we
saw previously:

```postgresql
DROP TABLE IF EXISTS users;

CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT NOT NULL,
  password TEXT NOT NULL
);
```

For example and demo purposes (you wouldn't do this in a real application in the
real world), let's insert a test user with a plain text password:

```postgresql
INSERT INTO users(id, username, password) VALUES(1, 'test', 'test');
```

Perform a `SELECT * FROM users;` query and look at the results:

![user-password-plaintext](https://curriculum-content.s3.amazonaws.com/spring-mod-2/security/user-password-plaintext.png)

As we can see, the plain text password we inserted is completely visible to us
as the developers. This means it would also be visible to hackers. If the
database is hacked, the hacker can see the values in the password column and use
those credentials for malicious activity such as:

1. Logging into this application with that username and password combination.
2. Using those credentials for other applications the user could use.

We recently just learned that hashing is the most secure technique to use in
order to store our passwords. So how can we do that?

We will be using the **pgcrypto extension** to help secure our passwords.

In the query tool, type the following to add the extension and then run the
query:

```postgresql
CREATE EXTENSION pgcrypto;
```

![create-extension-pgcrypto](https://curriculum-content.s3.amazonaws.com/spring-mod-2/security/create-extension-pgcrypto.png)

Now let's insert a new user into the database! But this time, we'll use the
`crypt` function to hash our password.

```postgresql
INSERT INTO users(id, username, password) VALUES(2, 'secureTest', crypt('test', gen_salt('bf')));
```

So what is the `crypt` function taking in as parameters? Well first, it will
take in the password in plain text that we want to put through a hashing 
algorithm. Then we need to specify the salt to use. The salt is used along with
hashing algorithms and will add a random piece of data prior to hashing. We'll
pass in `bf`, which will make use of the blowfish algorithm. The blowfish
algorithm is used with `bcrypt`, which is how we will be handling our
passwords in our Spring application thus far in our `PasswordEncoder` beans.

If we run the query and then run a `SELECT * FROM users;`, notice the password
is no longer in plain text!

![hashed-password-in-database](https://curriculum-content.s3.amazonaws.com/spring-mod-2/security/hashed-password-in-database.png)

As the developers, we have no idea now what the `secureTest` user's password
could be if we were just looking at the database. This means, a hacker will
also not be able to visibly see the password either. If values are hashed well,
then the hacker can't use that information to log in as the user.

So we might be thinking now, how can us, as the developers, confirm the password
in the application is valid? We'll cover that in the next lesson.

## Conclusion

In this lesson, we learned about the differences between encoding, encrypting,
and hashing as they are different techniques used for different purposes. We
also learned how to secure our passwords and sensitive data at rest in a
PostgreSQL database using the pcrypto extension.

## Resources

- [Spring Security Fundamentals - Lesson 1 - First Steps](https://youtu.be/nSu9ElsnNtY?t=2281)
- [The Difference Between Encoding, Encryption, and Hasing](https://medium.com/swlh/the-difference-between-encoding-encryption-and-hashing-878c606a7aff)
- [Encoding, Encryption, and Hashing](https://auth0.com/blog/encoding-encryption-hashing/)
- [Hashing in Action: Understanding bcrypt](https://auth0.com/blog/hashing-in-action-understanding-bcrypt/)
- [Security Encyclopedia: Salt](https://www.hypr.com/security-encyclopedia/salt#:~:text=A%20salt%20is%20a%20piece,passwords%20before%20they%20are%20stored.)
- [Storing Passwords Securely with PostgreSQL and pgcrypto](https://x-team.com/blog/storing-secure-passwords-with-postgresql/)
