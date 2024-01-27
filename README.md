#First, you'll need to install the schedule library in bash if you haven't already:
#   pip install schedule
#  Replace the placeholders like your_email@example.com, 
#   imap.example.com, your_search_string, 
#   and "C:\\Path\\To\\Save\\Folder" 
#   with your actual email address, IMAP server, search string, 
#   and the folder where you want to save the emails, respectively.

# Import necessary libraries
import imaplib
import email
import os
from email.header import decode_header
from getpass import getpass
import schedule
import time

# Function to decode email subject
def decode_subject(encoded_subject):
    # Decode the subject using the email library's decode_header function
    decoded, encoding = decode_header(encoded_subject)[0]

    # If the decoded subject is a bytes object, decode it to a string using the specified encoding or 'utf-8'
    if isinstance(decoded, bytes):
        decoded = decoded.decode(encoding if encoding else 'utf-8')

    # Return the decoded subject
    return decoded

# Function to save email to a specified folder
def save_email(email_message, save_folder):
    # Decode the subject of the email
    subject = decode_subject(email_message["Subject"])

    # Create a file name by combining the specified folder and the decoded subject
    file_name = os.path.join(save_folder, f"{subject}.eml")

    # Write the email message to a file with the computed file name
    with open(file_name, 'wb') as file:
        file.write(email_message.as_bytes())

# Function to handle email sorting and saving
def process_emails():
    # Your email configuration
    email_address = "your_email@example.com"
    email_password = getpass(prompt="Enter your email password: ")
    imap_server = "imap.example.com"
    imap_port = 993

    # String value to filter emails
    search_string = "your_search_string"

    # Folder to save the emails
    save_folder = "C:\\Path\\To\\Save\\Folder"

    # Connect to the IMAP server
    mail = imaplib.IMAP4_SSL(imap_server, imap_port)
    mail.login(email_address, email_password)

    # Select the mailbox you want to search (e.g., "inbox")
    mail.select("inbox")

    # Search for emails containing the specified string in the subject
    status, messages = mail.search(None, f'(SUBJECT "{search_string}")')

    # Check if the search operation was successful
    if status == 'OK':
        # Get the list of email IDs
        email_ids = messages[0].split()

        # Loop through each email ID
        for email_id in email_ids:
            # Fetch the email using its ID
            _, msg_data = mail.fetch(email_id, "(RFC822)")

            # Parse the email message from the fetched data
            email_message = email.message_from_bytes(msg_data[0][1])

            # Save the email to the specified folder
            save_email(email_message, save_folder)

    # Logout from the email server
    mail.logout()

    # Print a success message
    print("Emails sorted and saved successfully.")

# Schedule the script to run every day at the specified time (e.g., 2:00 AM)
schedule.every().day.at("02:00").do(process_emails)

# Run the scheduling loop
while True:
    # Check and run pending scheduled tasks
    schedule.run_pending()

    # Sleep for 1 second to avoid high CPU usage in the loop
    time.sleep(1)
