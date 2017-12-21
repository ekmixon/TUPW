# TUPW

Safely store credentials in config files

This program serves as an example of how to safely store credentials in config files. It works as a command line tool to encrypt and decrypt credentials. The decryption part can be incorporated into an application.

The idea is to store credentials in a config file in an encrypted form like this:

    <credentials>
      <user name="dbuser" user="1$cYKgznOQzmzMwAe72hc53Q==$Zg/U7gN4Q9TAz5jxnPxMWg==$r3bXCuG5vb5B3f+B0IUV+6bizLWI58fz2GkKc5dYFSA=" password="1$m6N9mBSNATy6AozEGPv+yw==$pVMq/bhCeTJ2MIMDQVo0nOTQ78HuiUOcUpweyX/KaK8=$tFSn2LNUgTPiThgf4TgJwtJn/MIt6ysVFtRO96G63JI="/>
    </credentials >

The encrypted data consists of four parts separated by '$' characters:

1. The format code: 1 =`{IV}{AES-128-CFB-ABytPadding}{HMAC}`
2. The IV
3. The AES-128-CFB-ABytPadding encrypted data
3. The HMAC of the format code, the IV and the encrypted data

So these data specify the value of the initialization vector used for encryption, the type of encryption ([AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard "AES") 128 bit in [CFB](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CFB "CFB") mode with [arbitrary tail byte ("ABytPadding") padding](https://eprint.iacr.org/2003/098.pdf "AByt-Pad"), and the [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code "HMAC") of all these values. But where does the encryption key come from?

The encryption key is generated by calculating the [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code "HMAC")-[SHA-256](https://en.wikipedia.org/wiki/SHA-2 "SHA-256") value of a file that is filled with random bytes. This file should be stored in a directory that is only accessible by the user that runs the application and nobody else.

The HMAC needs a key that is hard-coded in the program. So the key is calculated from something that is "known" (the HMAC key) and something that is in "posession" of the program (the file).

The above data can only be decrypted with the same file and the same HMAC key.

The program is used like this ('d:\keyfile.bin' is the name of the key file):

    java -jar tupw.jar encrypt d:\keyfile.bin dbUser

This generates (for example) the following output:

    Encryption = '1$cYKgznOQzmzMwAe72hc53Q==$Zg/U7gN4Q9TAz5jxnPxMWg==$r3bXCuG5vb5B3f+B0IUV+6bizLWI58fz2GkKc5dYFSA='

Note that the "iv" part (the one after '1$') of the encryption will change with each invocation of the program as it is derived from a secure random number generator and hence the result of the encryption (which uses the random iv) and also the HMAC will be different, as well, even if the same key file is used in all of these invocations.

Of course, one would need the keyfile to decrypt this like so:

    java -jar tupw.jar decrypt d:\keyfile.bin "1$cYKgznOQzmzMwAe72hc53Q==$Zg/U7gN4Q9TAz5jxnPxMWg==$r3bXCuG5vb5B3f+B0IUV+6bizLWI58fz2GkKc5dYFSA="

which yields (with the correct key file):

    Decryption = 'dbUser'
    
This way one can store the credentials and the key file in configuration management systems without storing them in the clear.

The decryption part of the program would typically be copied and used in an application to decrypt the credentials in the configuration file.

Of course, this is not perfectly safe, as an attacker can get access to the machine and extract the key file and the program classes and reverse engineer the way the key is calculated.

This program just makes it harder to get at the credentials, as both the file and the program code are needed to reconstruct the encryption key.

## Contributing

Feel free to submit a pull request with new features, improvements on tests or documentation and bug fixes.

## Contact

Frank Schwab ([Mail](mailto:frank.schwab@deutschebahn.com "Mail"))

## License

TUPW is released under the 3-clause BSD license. See "LICENSE" for details.
