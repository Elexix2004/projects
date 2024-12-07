#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void decryptPIN(char* encryptedPIN, char* decryptedPIN, char key) {
    // Bit rotate (right rotation by 3 bits)
    for (int i = 0; i < 5; i++) {
        decryptedPIN[i] = (((unsigned char)encryptedPIN[i] >> 3) | (encryptedPIN[i] << 5)); // Circular right shift by 3 bits
    }

    // XOR cipher
    for (int i = 0; i < 5; i++) {
        decryptedPIN[i] = decryptedPIN[i] ^ key;
    }

    decryptedPIN[5] = '\0'; // Null terminator to end the string
}

int main(void) {
    // Input the encrypted PIN as received from Arduino
    char inputEncryptedPIN[6];
    printf("Enter the encrypted PIN to decrypt: ");
    scanf("%5s", inputEncryptedPIN);

    // Decrypt the PIN
    char key = 'z';
    char decryptedPIN[6];
    decryptPIN(inputEncryptedPIN, decryptedPIN, key);

    printf("Decrypted PIN: %s\n", decryptedPIN);

    return 0;
}
