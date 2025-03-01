# Steganography
import cv2
import numpy as np

# Function to convert text to binary
def text_to_binary(text):
    return ''.join(format(ord(i), '08b') for i in text)

# Function to convert binary to text
def binary_to_text(binary_data):
    all_bytes = [binary_data[i:i+8] for i in range(0, len(binary_data), 8)]
    return ''.join(chr(int(byte, 2)) for byte in all_bytes if int(byte, 2) != 0)

# Function to hide text inside an image using LSB
def hide_text(image_path, text, output_path):
    image = cv2.imread(image_path)
    binary_text = text_to_binary(text) + '1111111111111110'  # Delimiter to mark end of message
    data_index = 0
    binary_length = len(binary_text)

    for row in image:
        for pixel in row:
            for channel in range(3):  # Iterate over BGR channels
                if data_index < binary_length:
                    pixel[channel] = (pixel[channel] & 254) | int(binary_text[data_index])
                    data_index += 1

    cv2.imwrite(output_path, image)
    print("Message hidden successfully in", output_path)

# Function to extract hidden text from stego image
def extract_text(stego_image_path):
    image = cv2.imread(stego_image_path)
    binary_data = ""

    for row in image:
        for pixel in row:
            for channel in range(3):
                binary_data += str(pixel[channel] & 1)

    delimiter = '1111111111111110'
    extracted_text_binary = binary_data.split(delimiter)[0]
    extracted_text = binary_to_text(extracted_text_binary)

    return extracted_text

# Function to calculate Mean Squared Error (MSE)
def calculate_mse(original, stego):
    mse = np.mean((original - stego) ** 2)
    return mse

# Function to calculate Peak Signal-to-Noise Ratio (PSNR)
def calculate_psnr(original, stego):
    mse = calculate_mse(original, stego)
    if mse == 0:
        return 100  # No difference at all
    max_pixel_value = 255.0
    psnr = 10 * np.log10((max_pixel_value ** 2) / mse)
    return psnr

# Main Execution
original_image_path = "wallpaperflare.com_wallpaper (1).jpg"  # Original image
stego_image_path = "after image.png"  # Stego image with hidden text
secret_message = "I love the last of us"  # Message to hide

# Hide message
hide_text(original_image_path, secret_message, stego_image_path)

# Extract hidden message
extracted_message = extract_text(stego_image_path)
print(" Extracted Message:", extracted_message)

# Compute PSNR & MSE
original_image = cv2.imread(original_image_path)
stego_image = cv2.imread(stego_image_path)

mse = calculate_mse(original_image, stego_image)
psnr = calculate_psnr(original_image, stego_image)

print(f" MSE (Mean Squared Error): {mse:.2f}")
print(f" PSNR (Peak Signal-to-Noise Ratio): {psnr:.2f} dB")
****
