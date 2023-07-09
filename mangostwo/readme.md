Creating a user:

the password is hashed as follows:
 select SHA1(CONCAT(UPPER('username'),':',UPPER('password')))