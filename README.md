[View Full Formal PDF Report](./Phishing%20Investigation%20Report.pdf)
# phishing-analysis-lab
# Phishing Email Investigation

To investigate a phishing email and its attachments, I set up a virtual machine in VMware to avoid potentially infecting my host machine.

I've installed the following tools on my VM and bookmarked the following sites on my VM’s web browser:

### Apps
* **Notepad++**: For analysing the email and its content.
* **HxD**: For analysing Hex-coded content.
* **7-Zip**: For extracting compressed files.
* **ExifTool**: For analysing the metadata of a file.

### Web
* **CyberChef**: To translate coded content to various other formats.
* **GaryKessler.net**: To analyse the file signatures of various file types.

---

## The Process

![Windows machine with tools running in VMware.](<images/Screenshot 1 Windows machine running with tools in VMWare..png>)

*Screenshot 1: Windows machine with tools running in VMware.*

After downloading a `.eml` file and opening it in Notepad++, it should be noted that `Return-Path` and `Reply-To` are completely different email addresses. This is suspicious and warrants further investigation.

![Return-Path and Reply-To.](<images/Screenshot 2 Return Path and Reply To..png>)

*Screenshot 2: Return-Path and Reply-To.*

An additional sign of suspicious activity is that the `Authentication-Results`, where we check the status of Sender Policy Framework (SPF), DMARC, and DKIM results, has been set to `fail`. This means that the domain of `microapple.com` does not recognise the IP `93.99.104.210` as an authorised sender of emails originating from the `microapple.com` domain.

![Authentication-Results.](<images/Screenshot 3 Authentication results.png>)

*Screenshot 3: Authentication-Results.*

I found an initial portion of content under the first boundary that has been encoded in Base64. I’ll use CyberChef to decode the content.

![Initial Base64 content.](<images/Screenshot 4 Base 64 content.png>)

*Screenshot 4: Initial Base64 content.*  

After using CyberChef to decode the Base64 encoded content, a message can be seen. The output of the message has been recorded as a `.txt` file.

![Decoded Base64 message recorded.](<images/Screenshot 5 decoded contact recorded..png>)

*Screenshot 5: Decoded Base64 message recorded.*

The decoded content makes reference to an attachment sent with the email. Further review in Notepad++ reveals another boundary that identifies the beginning of new content—an attachment identified as `PuzzleToCoCanDa.pdf`. To verify that this file is indeed a PDF, we copy the Base64 content directly below it into CyberChef and convert the Base64 encoding to Hex.

We do this because, by looking at the first two bytes of Hex code, we can compare this file’s signature (or magic number) against other known file signatures to confirm whether it is actually a PDF file.

![File signature captured.](<images/Screenshot 6 file signature captured.png>)

*Screenshot 6: File signature captured.*

After comparing the file signature of the alleged PDF file with a catalogue of known file signatures on the Gary Kessler File Signatures page, we can see that the signature actually matches that of a `.zip` file. This requires further investigation, so we encode the content back to Base64 and save the output as `Attachment.zip` to investigate further.

![File signature identified as a .zip file.](<images/Screenshot 7 signature identified as zip.png>)

*Screenshot 7: File signature identified as a .zip file.*

![Base64 content saved as Attachment.zip.](<images/Screenshot 8 Base 64 content saved as zip file.png>)

*Screenshot 8: Base64 content saved as Attachment.zip.*  

After extracting `Attachment.zip`, a directory opens, initially showing only two files. However, after selecting **View** and checking the **Hidden Items** box, a third document is identified. Two of the documents do not have a recognised file type, while the third file is appended with `.xlsx`, indicating that it could potentially be opened with Microsoft Excel.

![Zip file extracted.](<images/Screenshot 9 zip file extracted.png>)

*Screenshot 9: Zip file extracted.*   

We want to identify and confirm these file types. Since these three files cannot be viewed in their Hex format using CyberChef, we introduce a new tool by dragging and dropping them into HxD to reveal their Hex formats and compare their file signatures against the resource on Gary Kessler’s site.

![File types investigated by comparing signatures.](<images/Screenshot 10 file types investigated using signatures.png>)

*Screenshot 10: File types investigated by comparing signatures.*

![File types confirmed and renamed with correct extensions.](<images/Screenshot 11 File types confirmed.png>)

*Screenshot 11: File types confirmed and renamed with correct extensions.* 

Upon confirming the correct file types of the extracted documents, we are now able to open each document to investigate further:
* The `.jpeg` reveals an image but does not provide much value to the investigation.
* The `.pdf` refers to the location of the email sender and states that it can be found in the `.xlsx` document.

Due to the CTF nature of this phishing challenge, additional hidden info lies on the second sheet of this spreadsheet and can only be viewed after removing all formatting:

![First page of spreadsheet.](<images/Screenshot 12 First page of spreadsheet.png>)

 *Screenshot 12: First page of spreadsheet.*

![Apparently blank second page.](<images/Screenshot 13 Apparently blank second page.png>)

 *Screenshot 13: Apparently blank second page.*

![Removed formatting reveals Base64 encoded message.](<images/Screenshot 14 Base 64 encoded message revealed.png>)

*Screenshot 14: Removed formatting reveals Base64 encoded message.*
  
![Location of sender identified.](<images/Screenshot 15 Location of sender identified..png>)

*Screenshot 15: Location of sender identified.* 

Going back to our `.eml` file in Notepad++, we can see that the `Received: from localhost` field identifies `Emkei.cz` as the email service that sent the email. Upon further investigation, this service has been identified as a free fake email service available on the web.

![Fake email service identified.](<images/Screenshot 16 fake email service identified.png>)

*Screenshot 16: Fake email service identified.* 

The author of the email could be identified by using ExifTool to examine the metadata of the attachments downloaded from the email:

![Author identified by examining metadata.](<images/Screenshot 17 Author identified.png>)

*Screenshot 17: Author identified by examining metadata.* 

---

## Lessons Learned

When investigating an email, there are certain headers and fields that warrant close attention:

* **`Received:`**  
  These are the servers that received the email. We should perform OSINT on these and investigate the sender IP reputation, domain reputation, and the email service that sent the email.

* **`Return-Path:`**  
  This is the email address that will receive a failed delivery notice.

* **`Authentication-Results:`**  
  This is where we check the status of SPF, DMARC, and DKIM.

* **`Subject:`**  
  This is the title of the email. We can use this field to search across the entire organisation if an email security gateway or global search capability is available.

* **`From:` / `Reply-To:`**  
  When there is a discrepancy between these two fields, it warrants further investigation.

* **File Verification:**  
  File types must always be verified by checking their file signature (magic bytes) rather than relying solely on file extensions.

* **Environment Isolation:**  
  Always use a Virtual Machine (VM) to investigate suspicious emails and attachments. 
  Always use a Virtual Machine (VM) to investigate suspicious emails and attachments.
  Always use a Virtual Machine (VM) to investigate suspicious emails and attachments.
