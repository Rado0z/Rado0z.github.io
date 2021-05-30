---
title: Decrypting Signal DB for Android
published: true
categories: DFIR
---


## Introduction

Signal databases are encrypted in all operating system in different way. The easiest one is desktop application, it store the key in config.json file that you can easily decrypt it with sqlcipher.

For Android, we have to get three values in order to decrypt the database using AES-GCM mode, the first one is the key value which is USERKEY\_SignalSecret keystore. Second is the cipher text with IV values.

           

### Location

Windows:

**Database:** C:\\Users\\<username>\\AppData\\Roaming\\Signal\\sql\\db.sqlite

**Key:** C:\\Users\\Digisecure\\AppData\\Roaming\\Signal\\config.json

Android:

**Database:** /data/data/org.thoughtcrime.securesms/databases/signal.db

**key:** /data/keystore/user\_0/10044\_USRSKEY\_SignalSecret

**ciphertext with IV + authTag:** org.thoughtcrime.securesms\\shared\_prefs\\org.thoughtcrime.securesms\_preferences.xml

so in this blog we will foucs only on how to decrypt signal database in for Android.
	
### Android Decryption

Signal uses AES-GCM mode encryption method to encrypt the database using sqlcipher. First, it takes sql cipher key then use AES-GCM key from USERKEY + IV to encrypt the database, this values stored in **org.thoughtcrime.securesms\_preferences.xml.** in order to decrypt we should reverse it to get the sql cipher key.

In order to decrypt it we need three values:

1. decrypted \"app-id"\_USERKEY\_SignalSecret which is 16 byte starting from offset 2D to 3C.
	
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Signal_Keystore 1.png)
	
2. then will take the base64 encoded values from
	**org.thoughtcrime.securesms\_preferences.xml** the string name =pref\_database\_encrypted\_secret
	
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/secure_xml_preferences.png)
	
Then will take base64 data (cipher text) value convert hex like this: **864531f86b5b9dfad548f6e8c91712208d2d4477925240d586f1a3d825b545a2**246baf3b35662f5eb825dd85c9e39166
the last 32 character is the auth tag.
	
**Note**: if the developer uses online java AES-GCM code, maybe no need to split the auth tag from cipher text. But in my case with cyberchef I have to split them.
	
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/CyberChef.png)

3. now we got the decrypted key of signal database. the last step is how to open it with with sqlcipher.
	just quick look at the source code of android singal. they used these values in order to encrypted.
	
``` java
public final class SqlCipherDatabaseHook implements SQLiteDatabaseHook {

@Override

public void preKey(SQLiteDatabase db) {

db.rawExecSQL("PRAGMA cipher\_default\_kdf\_iter = 1;");

db.rawExecSQL("PRAGMA cipher\_default\_page\_size = 4096;");

}
	
```

we have to choose the passpharse with and put the key there. then choose the custom 
	

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/Deceypt_SignalDB 1.png)
	

and here we go !!!! :)
	
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/open_it.png)