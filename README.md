# WinRAR-Keygen
WinRAR is a trialware file archiver utility for Windows, developed by Eugene Roshal of win.rar GmbH.

It can create and view archives in RAR or ZIP file formats and unpack numerous archive file formats.

WinRAR is not a free software. If you want to use it, you should pay to RARLAB and then you will get a license file named "rarreg.key".

This repository will tell you how WinRAR license file "rarreg.key" is generated.

2. How is "rarreg.key" generated?
WinRAR uses a signature algorithm, which is a variant of Chinese SM2 digital signature algorithm, to process the user's name and the license type he/she got. Save the result to "rarreg.key" and add some header info, then a license file is generated.

The following will talk about the detail of the signature algorithm that WinRAR uses and how WinRAR process the user's name and his/her license type.

2.1. Signature Algorithm
WinRAR uses ECC (Elliptic-curve Cryptography) to do signature. The elliptic curve it uses is over a composite finite field FiniteField where the primitive polynomial of base finite field is:


And the primitive polynomial of extend finite field is:


The elliptic curve equation is:


Let G be base point that will be used during signature. The exact value is:

auto G = ecCurve.GetPoint({           // X, represents a polynomial 
  0x38CC, 0x052F, 0x2510, 0x45AA,   // over GF((2 ^ 15) ^ 17), the first term  
  0x1B89, 0x4468, 0x4882, 0x0D67,   // is in the front.
  0x4FEB, 0x55CE, 0x0025, 0x4CB7,
  0x0CC2, 0x59DC, 0x289E, 0x65E3,
  0x56FD
}, {                                  // Y, represents a polynomial 
  0x31A7, 0x65F2, 0x18C4, 0x3412,   // over GF((2 ^ 15) ^ 17), the first term  
  0x7388, 0x54C1, 0x539B, 0x4A02,   // is in the front.
  0x4D07, 0x12D6, 0x7911, 0x3B5E,
  0x4F0E, 0x216F, 0x2BF2, 0x1974,
  0x20DA
});
P is the order that will be used during signature. The exact value is:

const uint64_t P[4] = {   // P = 0x1026dd85081b82314691ced9bbec30547840e4bf72d8b5e0d258442bbcd31
    0x5e0d258442bbcd31,
    0x0547840e4bf72d8b,
    0x2314691ced9bbec3,
    0x0001026dd85081b8
};
2.1.1 The Generation of PrivateKey PrivateKey, Random Number K and Hash e
To generate PrivateKey PrivateKey, random number K and hash e, we need two byte-arrays. We call them in_data and in_private. Their length is in_data_length and in_private_length respectively.

PrivateKey is generated by in_private.
e is generated by in_data.
K is generated by both in_data and in_private.

For better understanding, here I give an case:

byte in_private[7] = { 'P', 'h', 'a', 'n', 't', 'o', 'm' }; 
byte in_data[9] = { 'L', 'a', 'b', 'y', 'r', 'i', 'n', 't', 'h' }; 
where in_private_length is 7 and in_data_length is 9.

2.1.1.1 The Generation of PrivateKey.
Calculate in_private's SHA-1 digest.

In the case I give, it should be:

byte in_private_sha1[] = {        //  standard sha-1 value is : {
    0xc0, 0x16, 0x3f, 0x02,       //      0x02, 0x3f, 0x16, 0xc0, 
    0x78, 0x4b, 0xac, 0x8c,       //      0x8c, 0xac, 0x4b, 0x78, 
    0xaf, 0x97, 0x0d, 0xe6,       //      0xe6, 0x0d, 0x97, 0xaf, 
    0xd9, 0x46, 0xa4, 0x91,       //      0x91, 0xa4, 0x46, 0xd9, 
    0xdc, 0xb3, 0xbf, 0x06        //      0x06, 0xbf, 0xb3, 0xdc
};                                //  }
NOTICE: Compared with standard SHA-1 calculation, WinRAR reversed each 32-bits block.

If in_private is null, use default value:

byte in_private_sha1[] = { 
    0x81, 0xb7, 0x3e, 0xeb, 
    0x29, 0x53, 0x26, 0x50, 
    0xa3, 0xf4, 0x5e, 0xdc, 
    0xd5, 0xb9, 0x47, 0x68, 
    0x4c, 0x3b, 0xe4, 0xcd 
};
It is probably the SHA-1 digest of some secret data used in RARLAB

To get each part of PrivateKey, do 15 rounds of SHA-1 calculation:

1. Let i be uint32_t varing from 1 to 15.

2. In each round, we calculate the SHA-1 digest of a 24-bytes-long byte-array which is the combination of i (little-endian) and src_data_sha1. Then take the first two bytes and append them at the end of PrivateKey.

After that, PrivateKey should be a 30-bytes-long and its value is:



In the case I give, PrivateKey is:

byte PrivateKey[30] = {       // PrivateKey = 0xb7e256217a67f14e3fb4246e889ea18b69b246616e04525e96d515831f2a  
    0x2a, 0x1f, 0x83, 0x15,   // which is a 240-bits-long integer.
    0xd5, 0x96, 0x5e, 0x52, 
    0x04, 0x6e, 0x61, 0x46, 
    0xb2, 0x69, 0x8b, 0xa1, 
    0x9e, 0x88, 0x6e, 0x24, 
    0xb4, 0x3f, 0x4e, 0xf1, 
    0x67, 0x7a, 0x21, 0x56, 
    0xe2, 0xb7 
};
In my code I use a uint64_t[4] array to store it.

2.1.1.2 The Generation of K and e.
During the generation of PrivateKey, there is a temporary 24-bytes-long byte-array. After all of the 15 rounds, the temporary byte-array should be the combination of uint32_t(15) (little endian) and src_data_sha1.

In the case I give, it should be:

byte in_private_sha1_temp[] = { 
    0x0f, 0x00, 0x00, 0x00, 
    0xc0, 0x16, 0x3f, 0x02,  
    0x78, 0x4b, 0xac, 0x8c,  
    0xaf, 0x97, 0x0d, 0xe6,  
    0xd9, 0x46, 0xa4, 0x91, 
    0xdc, 0xb3, 0xbf, 0x06 
};
Calculate in_data's SHA-1 digest:

In the case I give, it should be:

byte in_data_sha1[] = { 
    0xfc, 0x0e, 0xcd, 0xba, 
    0x12, 0xa0, 0xc6, 0x2e, 
    0x96, 0xf6, 0xbe, 0xdb, 
    0x9f, 0x89, 0x72, 0x10, 
    0x05, 0x05, 0x71, 0xe0 
};
Append

byte empty_sha1[] = { 
    0x43, 0x8d, 0xfd, 0x0f, 
    0x7c, 0x3c, 0xe3, 0xb4, 
    0xd1, 0x1b, 0x46, 0x53, 
    0x46, 0xa5, 0x27, 0x0f, 
    0x0d, 0xd9, 0x50, 0x10
};
at the end of in_data_sha1 so we can get byte-array (in_data_sha1 + empty_sha1). The byte-array appended is the SHA-1 digest of null while 5 SHA-1 initial constants is set 0. Then take the first 30 bytes of (in_data_sha1 + empty_sha1) as e.

Append (in_data_sha1 + empty_sha1) at the end of in_private_sha1_temp.

In the case I give, it should be:

byte in_private_sha1_temp[] = { 
    0x0f, 0x00, 0x00, 0x00,   // original in_private_sha1_temp.
    0xc0, 0x16, 0x3f, 0x02,  
    0x78, 0x4b, 0xac, 0x8c,  
    0xaf, 0x97, 0x0d, 0xe6,  
    0xd9, 0x46, 0xa4, 0x91, 
    0xdc, 0xb3, 0xbf, 0x06, 
    0xfc, 0x0e, 0xcd, 0xba,   // in_data_sha1
    0x12, 0xa0, 0xc6, 0x2e, 
    0x96, 0xf6, 0xbe, 0xdb, 
    0x9f, 0x89, 0x72, 0x10, 
    0x05, 0x05, 0x71, 0xe0,
    0x43, 0x8d, 0xfd, 0x0f,   // empty_sha1
    0x7c, 0x3c, 0xe3, 0xb4, 
    0xd1, 0x1b, 0x46, 0x53, 
    0x46, 0xa5, 0x27, 0x0f, 
    0x0d, 0xd9, 0x50, 0x10
};
To get each part of red_K, do 15 rounds of SHA-1 calculation.
In each round, do

++*reinterpreter_cast<uint32_t*>(in_private_sha1_temp);
first, then calculate the SHA-1 digest of in_private_sha1_temp and append the first two bytes of the digest at the end of K.

In the case I give, it should be:

byte K[] = { 
    0xeb, 0xed, 0x4f, 0xba, 
    0x0b, 0x30, 0xe8, 0x26, 
    0xf4, 0xec, 0xe6, 0x92, 
    0x76, 0xcc, 0xe8, 0x0b, 
    0xa8, 0x9c, 0x6f, 0x3a, 
    0x41, 0x6d, 0x5c, 0xfe, 
    0x21, 0x42, 0x5a, 0x5a, 
    0x5d, 0xbe
};
In my code I use a uint64_t[4] array to store it.

2.1.2 The Generation of Singnature rs and PublicKey PublicKey
Now we have PrivateKey PrivateKey, random number K, hash e, order P and base point G.

equation

NOTICE:

1. The dot in equation refers to the elliptic curve point multiplication on equation over FiniteField.

2. equation means takeing X-axis value of a point. This value is a polynomial over FiniteField.

3. Function T converts a polynomial over FiniteField to a integer whose bit length would not larger than 15 * 17 = 255. The detail of function T will be talked about later.

4. equation is integer addition.

equation

NOTICE:

1. equation is integer multiplication.

equation

NOTICE:

1. equation is division over FiniteField.

2. equation is binary AND operation.

About function T:
As I said before, it converts a polynomial over FiniteField to a integer. If I use a 17-elements-long list represent a polynomial over FiniteField, whose every element represents a polynomial over equation, function T can be defined as the following Python code:

def T(x : list):
    ret = 0
    for i in range(0, 17):
        ret += x[i] * 2 ** (15 * i)
    
    return ret
2.2. The Generation of "rarreg.key"
rarreg.key consists of a header, user's name, license type, UID, registration data and checksum.

2.2.1 Header
It is just a text line:

RAR registration data

Actually, when WinRAR verifies user's license file, it does not care what content the header have.

2.2.2 User's name
It is also just a text line. Here I give a case:

Phantom

2.2.3 License type
Also just a text line. Actually it can be any text. Here I give a case:

Single PC usage License

2.2.5 UID
It is just a join of two parts of registration data. Here I give a case:

UID=294d3fd81ae79b20c48c

Actually, when WinRAR verifies user's license file, it does not case UID at all.

2.2.5 Registration data
Registration data has four parts. We name them RegData0, RegData1, RegData2, RegData3 respectively.

If ECC signature of license type is r and s (, which means in_data = license type, in_private = null) , convert their value to hex string str_R, str_S. The max possible length of str_R and str_S is 60 because r and s are both 30 bytes long. So RegData1 is the format output of "60", str_S and str_R:

_stprintf_s(RegData[1], TEXT("60%060s%060s"), str_S, str_R);
In the case I give, it should be:

char RegData[1] = "60cc0b2d34fdb287124c6ca6b4f3239d36aa3ef82a9aba3a22d45552465c93260c3f609e601c26f312c4f44e7f8773a98b078809297303f8cac4a5ff92";
Let str_Kpub be the hex string of PublicKey that is generated by user's name (in_private = user's name). RegData3 is:

_stprintf_s(RegData[3], TEXT("%zd%.48s"), strlen(str_Kpub) - 4, str_Kpub);
In the case I give, it should be:

char RegData[3] = "6067c38462f2ff189c4b04e3c0540548d096e1068d9235023c";
Let str_Kpub2 be the hex string of PublicKey that is generated by RegData3 (in_private = RegData3). So:

_stprintf_s(RegData[0], TEXT("%s"), str_Kpub2);
In the case I give, it should be:

char RegData[0] = "c48c1c168f2ddf266ab3c33118678236ddeb1319b06790c929a038e487c79c60";
UID is:

_stprintf_s(UID, TEXT("UID=%.16s%.4s"), str_Kpub + 48, str_Kpub2);
In the case I give, it should be:

char UID = "UID=294d3fd81ae79b20c48c";
which is the same as what said before.

If ECC signature of str_UserName_RegData0 that is the join of user's name and RegData0 is r and s (, which means in_data = user's name + RegData0, in_private = null) , convert their value to hex string str_R2, str_S2. So RegData2 is the format output of "60", str_S2 and str_R2:

_stprintf_s(RegData[2], TEXT("60%060s%060s"), str_S2, str_R2);
In the case I give, it should be:

char RegData[2] = "60625690ab228461b54ab9a4c0a4cc5f7fdf47aa192f839596b1628abf1bec0f405163f46ce6a778adc398cefae18dbb0e4f46291e4d61e573ae0eaeb9";
2.2.5 Registration data
TODO
The output:
  _tprintf_s(TEXT("RAR registration data\n%s\n%s\n%s\n"), 
         UserName,
         LicenseType,
         UID);
  
  char temp[1024] = { };
  _stprintf_s(temp, TEXT("%zd%zd%zd%zd%s%s%s%s%lu"),
          strlen(RegisterData[0]),
          strlen(RegisterData[1]),
          strlen(RegisterData[2]),
          strlen(RegisterData[3]),
          RegisterData[0],
          RegisterData[1],
          RegisterData[2],
          RegisterData[3], 
          checksum);

  for (int i = 0; i < 8; ++i)
      _tprintf_s(TEXT("%.54s\n"), temp + i * 54);
is rarreg.key.
3. How to build keygen?
Please make sure that you have Visual Studio 2015 or the higher. Because this is a VS2015 project.

You can open the project by VS2015 IDE and then build it with one click. Or use VS Developer Command Prompt:

msbuild winrar-keygen.vcxproj /p:Configuration=Release;Platform=<x86|x64>;OutputPath=<your_output_dir> /t:build
NOTICE: Do not use "Debug" configuration. Otherwise you may approximately wait for half a minute every time you generate a license file.

4. Example
D:\Github\winrar-keygen>msbuild winrar-keygen.vcxproj /p:Configuration=Release;Platform=x64;OutDir=D:\winrar-keygen\ /t:build
    ......
    ......

D:\Github\winrar-keygen>cd D:\winrar-keygen

D:\winrar-keygen>winrar-keygen.exe "Phantom" "Single PC License" >> rarreg.key
