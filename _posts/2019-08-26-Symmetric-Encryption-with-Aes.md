There are many [symmetric encryption algorithms](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.symmetricalgorithm?view=netframework-4.8){:target="_blank"} in .NET world. I created a [demo project](https://github.com/dujushi/SymmetricEncryption){:target="_blank"} with [Aes](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.aes?view=netframework-4.8){:target="_blank"}. 

## Secret Key and Initialization Vector 
{% highlight csharp %}
public SymmetricEncryption(string secretKey, string initializationVector)
{
    _secretKey = secretKey;
    _initializationVector = initializationVector;
}
{% endhighlight %}

A secret key and an initialization vector are required for the data encryption. The initializaion vector is used to prevent repetition in the encrypted text. The lengths of secret key and initialization vector depend on the value of key size and block size. They are validated by [SymmetricAlgorithm abstract class](https://github.com/dotnet/corefx/blob/master/src/System.Security.Cryptography.Primitives/src/System/Security/Cryptography/SymmetricAlgorithm.cs#L59#L101){:target="_blank"}. This project uses the default setting. Secret key is of 32 characters and initialization vector is of 16 characters. You can use [RAMDOM.ORG](https://www.random.org/strings){:target="_blank"} to generate them.

## EncryptAsync
{% highlight csharp %}
public async Task<string> EncryptAsync(string plainText)
{
    using (var aes = Aes.Create())
    {
        SetKeyAndIV(aes);

        var encryptor = aes.CreateEncryptor();
        using (var memoryStream = new MemoryStream())
        {
            using (var cryptoStream = new CryptoStream(memoryStream, encryptor, CryptoStreamMode.Write))
            {
                using (var streamWriter = new StreamWriter(cryptoStream))
                {
                    await streamWriter.WriteAsync(plainText);
                }

                var encryptedArray = memoryStream.ToArray();
                var cipherText = Convert.ToBase64String(encryptedArray);
                return cipherText;
            }
        }
    }
}
{% endhighlight %}

The cipher text is Base64 encoded.

## DecryptAsync
{% highlight csharp %}
public async Task<string> DecryptAsync(string cipherText)
{
    using (var aes = Aes.Create())
    {
        SetKeyAndIV(aes);

        var decryptor = aes.CreateDecryptor();
        var cipherTextBytes = Convert.FromBase64String(cipherText);
        using (var memoryStream = new MemoryStream(cipherTextBytes))
        {
            using (var cryptoStream = new CryptoStream(memoryStream, decryptor, CryptoStreamMode.Read))
            {
                using (var streamReader = new StreamReader(cryptoStream))
                {
                    var plainText = await streamReader.ReadToEndAsync();
                    return plainText;
                }
            }
        }
    }
}
{% endhighlight %}

The cipher text must be Base64 encoded from the above method.