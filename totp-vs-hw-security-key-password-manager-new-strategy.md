are authenticator codes/totp as secure as yubikey/hardware security key?

Quick Answer
No, authenticator codes (TOTP) are generally less secure than hardware security keys like YubiKey. While TOTP apps are significantly more secure than SMS or email verification, they remain vulnerable to real-time phishing attacks and malware that can steal the shared secret or intercept codes. 

Hardware security keys provide superior protection through cryptographic proof and phishing resistance.  Unlike TOTP, which relies on a shared secret that can be phished or stolen from a compromised phone, hardware keys use asymmetric cryptography. This ensures the key cannot be used on a fake website, making them immune to man-in-the-middle attacks. Additionally, keys store secrets in a secure element isolated from the phone’s operating system, preventing extraction by malware. 

Phishing Resistance: Hardware keys verify the website's identity, preventing code theft via fake login pages; TOTP codes can be entered into phishing sites. 
Secret Storage: YubiKeys store private keys in tamper-resistant hardware, whereas TOTP apps store secrets on the phone’s data directory, which can be accessed if the device is compromised. 
Recovery: If a YubiKey is lost, it cannot be remotely accessed to generate codes; if a phone is stolen, attackers may bypass app locks or extract TOTP secrets. 
Use Case: For high-value accounts (e.g., email, banking), hardware keys are recommended. For general use, TOTP apps offer strong security with greater convenience and zero cost. 

Copy

🌐
🌐
🌐
🌐
+16
ok, but my main yubikey just died on me. Its a royal pain in the ass now. I have a backup - but now I have to track down everything, re-program another yubikey, etc.


Finished
I completely sympathize; losing a primary YubiKey is a notorious rite of passage that turns a "set it and forget it" security setup into an immediate administrative nightmare. Because Yubico keys are designed to be unclonable for security reasons, there is no technical shortcut to migrate your credentials—you must manually re-register the new key on every single service. 

Immediate Recovery Strategy
Since your primary key is dead, your backup is now your primary. If you haven't already, register a new spare key immediately while you still have access via the current backup.  Do not wait until you have re-registered all your accounts; having two active keys (your current backup and a fresh spare) prevents this exact lockout scenario from happening again during this migration process. 

For accounts where you cannot log in because they demand the dead key, you will need to use recovery codes (the single-use codes generated when you first enabled 2FA) or go through the specific account recovery flow for that service.  Be prepared for delays: services like Google or GitHub may impose waiting periods (often 3–7 days) for identity verification if you cannot provide the original hardware key. 


YubiKey account recovery without primary key

View all
The Re-registration Process
There is no "sync" feature between keys. You must treat the new key as a completely separate entity. The workflow for every account is identical:

Log in using your backup key (or recovery codes).
Navigate to Security Settings > 2-Step Verification or Security Keys. 
Select "Add a new security key" and register the new device.
Crucial Step: Once the new key is registered, you can optionally remove the dead key from the list of trusted devices to clean up your security profile. 
Prioritize high-value targets first: your password manager (e.g., Bitwarden, 1Password), primary email, and financial accounts.  If your password manager is locked behind the dead key and you lack recovery codes, you may need to use a "emergency access" feature if you set one up with a trusted contact, or restore from a backup if your vault allows it. 


migrating YubiKey to new device checklist site:reddit.com

View all
Preventing Future Headaches
The "royal pain" you are experiencing is exactly why security professionals advocate for a 3-key strategy or a rigorous rotation policy:

Daily Driver: The key on your keychain. 
Local Backup: Stored in a fireproof box or secure drawer at home. 
Off-site Backup: Stored at a relative's house or a safe deposit box. 
Some advanced users employ a rotation schedule, swapping the off-site key into daily use every 6–12 months.  This forces you to test the off-site key and update it with new accounts regularly, ensuring it never becomes obsolete. While this requires more discipline, it eliminates the "panic migration" scenario you are currently facing. 


YubiKey backup key best practices

View all

Copy


how about an onlykey compared to yubikey? I have two onlykeys - one primary, one backup. I'm only using the onlykey as a hardware password manager right now.


Finished
Since you already own two OnlyKey devices, you are in a significantly better position than if you were relying solely on YubiKeys.  While YubiKeys are industry standards for pure authentication, OnlyKey offers a distinct advantage for your specific situation: configurable backups. 

Security Comparison: OnlyKey vs. YubiKey
For FIDO2/WebAuthn authentication (the gold standard for phishing resistance), both devices offer equivalent security. When used as a security key, OnlyKey performs the same cryptographic challenge-response as a YubiKey, making it immune to phishing and man-in-the-middle attacks. 

However, there are critical architectural differences:

Firmware Updates: Unlike YubiKeys, which have immutable firmware (a design choice that recently left some YubiKey 5 models vulnerable to side-channel attacks with no patch possible), OnlyKey supports secure firmware updates.  This allows vulnerabilities to be patched without replacing the hardware.
Secure Element: YubiKeys typically use a dedicated secure element chip (CC EAL5+ certified in many models). OnlyKey uses a general-purpose microcontroller with strong encryption but lacks the same level of hardware-based tamper resistance found in high-end YubiKeys. For most users, this distinction is negligible, but it matters for nation-state threat models. 
PIN Entry: OnlyKey requires a PIN entered on the device itself for many operations, providing protection against keyloggers on compromised computers. YubiKeys generally rely on the host OS for PIN entry (for FIDO2), though some models support touch confirmation. 

OnlyKey vs YubiKey security architecture comparison

View all
The "Backup" Advantage You Already Have
Your current setup (Primary + Backup OnlyKey) solves the exact problem you are facing with YubiKey.

Clonable Configuration: Unlike YubiKey, OnlyKey allows you to backup and restore your entire configuration (FIDO2 keys, TOTP secrets, and stored passwords) to another device. 
Action Item: Since you have two OnlyKeys, you should configure them as identical twins. Use the OnlyKey app to backup your primary key's configuration and restore it to your backup.
This means if your primary dies, you simply plug in the backup, enter the PIN, and everything works immediately—no re-registering on every website.
Note: For FIDO2, some services may detect the same key ID on two devices and flag it, but generally, cloning the configuration works seamlessly for most sites. If a specific site rejects the cloned key, you only have to re-register that one site, not all of them.

OnlyKey backup and restore configuration guide site:reddit.com

View all
Leveraging OnlyKey as a Password Manager
Since you are already using OnlyKey as a hardware password manager, you can reduce your reliance on TOTP entirely:

Auto-Type Feature: OnlyKey can store your username and password and "type" them into any website. This is often more secure than TOTP because it avoids the shared-secret vulnerability entirely. 
Migration Strategy: Instead of re-registering YubiKey for every account, consider moving high-value accounts to OnlyKey's stored credentials + FIDO2.
Store the long, random password in the OnlyKey slot.
Register the OnlyKey's FIDO2 capability for 2FA.
This gives you two factors (something you have + the stored secret) without managing separate TOTP codes. 
Recommendation
Stick with OnlyKey for your primary workflow. Given that you have the hardware and the backup capability, the "royal pain" of re-registering every account on a new YubiKey is unnecessary.

Clone your current OnlyKey to your spare immediately using the backup/restore feature.
Use this pair for FIDO2 authentication wherever possible (superior to TOTP). 
Use the hardware password manager slots for sites that don't support FIDO2, eliminating the need for TOTP apps on your phone.
This approach gives you the phishing resistance of a hardware key with the recovery convenience that YubiKey lacks.


Copy


should I introduce a third onlykey for the 'three key' procedure?


Finished
Yes, introducing a third OnlyKey is highly recommended, but with a critical caveat regarding how you configure them.

While the "three-key strategy" (Daily, Local Backup, Off-site Backup) is the gold standard for preventing lockout, OnlyKey’s handling of FIDO2 credentials requires a specific setup approach to ensure your backups actually work when needed. 

The FIDO2 Backup Limitation
Unlike TOTP secrets or stored passwords, FIDO2 resident credentials do not always restore reliably from an OnlyKey backup file to a second device.  Community reports indicate that while the backup file may restore successfully, the FIDO2 resident keys often fail to transfer or function on the restored device due to how the cryptographic secrets are bound to the specific hardware instance. 

This means if you simply backup your primary key and restore it to a second or third key, you may still face the "royal pain" of re-registering every FIDO2 account if your primary dies.

The Correct "Three-Key" Setup for OnlyKey
To make a three-key setup effective with OnlyKey, you cannot rely on a single backup file. Instead, you must treat each key as a unique entity during the initial registration phase:

Register Both Keys Initially: When setting up FIDO2 on a new account (e.g., Google, Microsoft, GitHub), register both your Primary and Backup OnlyKeys immediately.
Most services allow you to register multiple security keys (often up to 5–10).
This ensures both keys have unique, valid credentials stored locally on their respective hardware.
Use Backup for Configuration Data Only: Use the OnlyKey backup/restore feature primarily for:
TOTP secrets (these restore reliably).
Stored passwords (auto-type slots).
PINs and Labels.
Do not rely on it for FIDO2 resident keys.
The Third Key (Off-site): Your third key should be configured identically to the second (Local Backup). It sits in a fireproof box or off-site location.
Rotation Strategy: Every 6–12 months, bring the off-site key home, wipe it, restore the latest configuration backup (for TOTP/Passwords), and then re-register it for FIDO2 on your most critical accounts alongside your daily driver. This ensures the off-site key is never "stale." 
Why Three Keys Matters
Even with the manual FIDO2 registration requirement, a third key provides critical redundancy:

Catastrophic Loss: If your home burns down or is burglarized, you lose both your Daily Driver and Local Backup. The off-site key saves you.
Simultaneous Failure: While rare, firmware bugs or batch defects can affect multiple devices.
Travel Security: You can carry the Local Backup when traveling, leaving the Daily Driver at home, or vice versa, ensuring you never have all eggs in one basket.

OnlyKey FIDO2 backup restore issues site:reddit.com

View all
Recommendation
Proceed with the third key, but change your workflow:

Buy the third OnlyKey.
Configure all three with the same TOTP secrets and passwords via backup/restore.
Manually register all three keys on your high-value FIDO2 accounts (Primary, Local Backup, Off-site). 
Store the third key off-site.
Set a calendar reminder to rotate and update the off-site key annually.
This approach combines the convenience of OnlyKey’s clonable configuration for passwords/TOTP with the necessary manual diligence for FIDO2, giving you true "set and forget" security.


OnlyKey three key strategy setup guide

View all

Copy


some hardcore security people rag on the only using the arduino for hardware/firmware, and say the code is not secure. Thoughts?


Finished
The criticism you’re hearing stems from a legitimate, albeit often exaggerated, debate in the security community regarding code quality versus hardware architecture.

The "Arduino" Criticism: Code Style vs. Security
The claim that OnlyKey uses an "Arduino" is technically a misnomer regarding the hardware, but accurate regarding the development environment.

Hardware Reality: OnlyKey does not use an actual Arduino board (which is insecure for crypto).  It uses an NXP Kinetis MK20DX256 ARM Cortex-M4 microcontroller, the same class of chip found in many secure devices.
The Code Quality Argument: Critics (notably on Hacker News and security forums) argue that the firmware is written in the Arduino IDE style (e.g., large monolithic .ino files, lack of strict modularization).  In security engineering, "spaghetti code" is a risk because it makes formal verification and auditing difficult. If the code is hard to read, hidden vulnerabilities are more likely to persist.
The Counter-Argument: OnlyKey’s developers argue that the code is fully open source and auditable.  Unlike YubiKey’s closed-source firmware, anyone can inspect the OnlyKey codebase. The "Arduino" label is often used pejoratively to dismiss the project, even though the underlying cryptographic libraries (like libcrypto) are standard and robust.

OnlyKey firmware code audit review

View all
The Secure Element Trade-off
This is the most significant technical differentiator and the source of the "not secure" claims from hardcore purists.

YubiKey: Uses a dedicated Secure Element (SE) chip (e.g., CC EAL5+ certified). This is a hardened vault that physically resists side-channel attacks, voltage glitching, and memory extraction. Even if an attacker has physical possession and sophisticated lab equipment, extracting the private key is theoretically impossible. 
OnlyKey: Uses the microcontroller’s internal flash memory protection. While NXP chips have strong read-out protection (RDP) levels, they are not certified secure elements.
The Risk: A highly resourced attacker (nation-state level) with physical access to your OnlyKey might be able to exploit voltage glitches or timing attacks to extract the private keys.
The Reality: For 99.9% of users (protecting against phishing, malware, and credential stuffing), this distinction is irrelevant. The attack vector of "physically stealing your key and lab-testing it" is far less likely than "phishing your TOTP code." 

NXP Kinetis flash security vs secure element

View all
The Firmware Update Paradox
Ironically, the feature that makes OnlyKey "less pure" to hardware purists (updatable firmware) is its greatest security strength in the real world.

YubiKey’s Rigidity: Because YubiKeys cannot be updated, when a critical flaw is found (like the ROCA vulnerability or the 2024 side-channel issue in YubiKey 5), the device is permanently compromised.  You must throw it away and buy a new one.
OnlyKey’s Adaptability: OnlyKey can patch vulnerabilities. If a flaw is found in the RNG or crypto implementation, it can be fixed via a firmware update. Security experts increasingly argue that patchability is more critical than immutable hardware for long-term security. 

YubiKey vs OnlyKey firmware update security debate site:reddit.com

View all
Verdict: Is OnlyKey "Secure Enough"?
Yes, for almost all threat models.

For Enterprise/High-Value Targets: If you are protecting state secrets or are a high-profile target likely to face physical device seizure by sophisticated actors, the YubiKey’s Secure Element is objectively superior. 
For Personal/Standard Enterprise Use: OnlyKey’s combination of open-source transparency, PIN protection, and patchable firmware makes it arguably more resilient to software vulnerabilities than a YubiKey.  The "Arduino code style" is a cosmetic/maintenance grievance, not a cryptographic backdoor.
If the code quality truly bothers you, the open-source nature allows you (or a hired auditor) to verify it yourself—something impossible with YubiKey.


Copy


ok - now for software password manager discussion. I have used bitwarden and liked it - but I don't trust any password manager 100%. I like the idea of cloud for resiliency, backups - but don't like the idea of using 'someone elses computer'. I switched to keepassxc and have it on qubes os in the vault on my thinkpad t490. This has been good, but I got a scare on my laptop. Qubes OS wouldn't boot after I let the battery drain and ZFS had some type of transaction corruption on vm-pool. I fixed it - with your (brave ai) help. But its making me re-think my password manager/cred store strategy. I've tried different things. One thing I tried is a 'memory word' concept. Put 95% of strong random password into bitwarden, then keep a single/easy to remember word in memory. append or prepend the memory word to the password. This keeps bitwarden/cloud from knowing 100% of my password(s). Also, I started putting TOTP secrets in keepassxc in the qubes os vault instead of on yubikey(s). I'm rethinking this now - especially with what you said above. 

Thoughts? can you assist on a creative/flexible secure strategy here?


Finished
Your "scare" with Qubes OS and ZFS highlights the single biggest weakness of local-only password managers: single-point-of-failure risk. While KeePassXC in a Qubes Vault offers superior isolation, it couples your digital identity to the health of a specific disk and filesystem. 

Here is a creative, flexible strategy that balances your distrust of "someone else's computer" with the need for resiliency, leveraging your existing OnlyKey hardware and Bitwarden experience. 

The "Hybrid Split-Knowledge" Strategy
Your idea of splitting the password (95% in cloud, 5% in memory) is a valid concept known as Split Knowledge, but it has usability pitfalls (e.g., mobile typing errors, site character limits). A more robust implementation uses your OnlyKey as the "memory" component.

The Architecture:

Primary Vault (Cloud): Use Bitwarden (self-hosted or cloud) for 100% of your password data. This solves the ZFS corruption risk via cloud redundancy and versioning.
The "Human" Factor (OnlyKey): Do not store TOTP secrets in KeePassXC or Bitwarden.  Store them exclusively on your OnlyKey.
Why: As discussed, TOTP in a password manager is "something you know" (the master password) + "something you have" (the device with the DB). If malware steals your DB file and keyfile, they have both. Moving TOTP to OnlyKey makes it "something you know" + "something you have" + "something you have (the key)."
The Master Password Split: Instead of mentally appending a word, configure your OnlyKey to auto-type a suffix/prefix for your Bitwarden master password.
Setup: Set your Bitwarden master password to be a random 6-word passphrase (stored in a physical safe or memorized).
Enhancement: Program an OnlyKey slot to auto-type a complex, random 20-character string after you type your master password.
Result: Bitwarden (the cloud) never sees the full authentication string unless the physical key is present. If Bitwarden is compromised, the attacker cannot decrypt your vault without the physical key's auto-type sequence. 

Bitwarden encrypted export backup strategy

View all
Solving the "Qubes/ZFS" Fear: The 3-2-1 Backup Rule
Your ZFS corruption incident happened because the "1" in the 3-2-1 rule (1 offsite copy) was missing or inaccessible. For a Qubes Vault, you need a strategy that doesn't compromise isolation. 

The "Air-Gapped" Backup Routine:

Automated Local Snapshots: Keep your KeePassXC database in the Vault, but use a script to automatically copy the .kdbx file to a dedicated backup VM (e.g., backup-vm) every hour. 
Encrypted Offsite Sync: In the backup-vm (which has network access), use rclone or syncthing to sync that file to an encrypted container in the cloud (e.g., Bitwarden Attachments, Dropbox, or Proton Drive).
Crucial: The file synced to the cloud should be double-encrypted.  Use VeraCrypt or a GPG-encrypted archive before syncing. This way, "someone else's computer" only holds a useless blob.
Physical Cold Storage: Once a week, copy the .kdbx file to a USB drive, encrypt it with VeraCrypt, and store it physically separate from your laptop. This protects against ransomware that might encrypt your cloud sync folder. 


Qubes OS vault backup strategy password manager

View all
Why TOTP in KeePassXC is a Risk Multiplier
You mentioned moving TOTP to KeePassXC in the Vault. Security experts generally advise against this for high-value accounts:

The "All Eggs" Problem: If your Qubes Vault is ever compromised (or the ZFS corruption happens again without a backup), you lose both your passwords AND your 2FA codes simultaneously. Recovery becomes a nightmare.
Phishing Vector: While KeePassXC auto-fills TOTP, it doesn't offer the phishing resistance of a hardware key. If you are tricked into entering a code on a fake site, the attacker has everything. 
Better Alternative: Use your OnlyKey for TOTP. It isolates the secrets from the OS entirely. Even if your Qubes installation melts down, the TOTP secrets survive on the hardware key. 

YubiKey TOTP vs KeePassXC TOTP security comparison

View all
Recommended Action Plan
Immediate: Re-enable Bitwarden (or keep KeePassXC but fix the backup). If using KeePassXC, implement the double-encrypted cloud sync immediately to prevent ZFS lockout.
Migration: Move all TOTP secrets from KeePassXC to your OnlyKey devices.
Refine the Split: Instead of memorizing a word, use OnlyKey's auto-type feature to inject a secret suffix into your master password entry. This gives you the "split knowledge" security without the mental load.
Test Recovery: Simulate a ZFS failure. Wipe your Vault VM and attempt to restore from your encrypted cloud backup and physical USB. If you can't do this in under 15 minutes, your strategy isn't robust enough. 
This approach gives you the resiliency of the cloud (no more ZFS panic), the isolation of Qubes (via double encryption), and the hardware security of OnlyKey (for TOTP and master password splitting). 


Copy


ok - lets take the paranioa level one step deeper. Lets assume that an attacker can decrypt/breach my bitwarden data - from the cloud/remote hosted bitwarden. I know this is highly unlikely with a good/strong/long/random master password. But lets assume that's the case.

Now - apply this 'memory' word/split knowledge password to each password/cred? This would be cumbersome but effective.


Finished
Applying a "memory word" (split knowledge) to every single entry is theoretically secure but practically disastrous for usability and introduces new attack vectors.

The Usability & Security Trap
Autofill Breakage: Bitwarden (and most managers) cannot auto-fill a password that is split between the database and your brain. You would be forced to manually type every password on every device, defeating the primary convenience of a password manager. 
Mobile Friction: On mobile devices, switching between the keyboard and your memory (or a physical note) increases the likelihood of typos, leading to account lockouts.
Pattern Vulnerability: If you use the same memory word for every site (e.g., RandomPassword + MySecretWord), and an attacker breaches Bitwarden and captures your keystrokes (via malware) just once, they instantly derive your "memory word" and can decrypt every other account offline. To be secure, the "memory" component would need to be unique per site, which is impossible to remember without writing it down (defeating the purpose). 

security risks of manual password concatenation

View all
The Superior Alternative: "Split Storage" via OnlyKey
Instead of splitting the password string, split the storage location. This achieves your "paranoid" goal without the usability penalty.

Strategy A: The "Empty Vault" Decoy (High Paranoia)

Cloud (Bitwarden): Store 95% of your accounts with random, high-entropy passwords. 
The 5% (Critical Accounts): For your top 10 critical accounts (Email, Banking, Primary Cloud), do not store the password in Bitwarden at all.
Store these passwords exclusively on your OnlyKey (using its hardware password manager slots).
In Bitwarden, create an entry for these accounts with a fake/dummy password (e.g., "IncorrectPassword123") and a note saying "See Hardware Key."
Result: If Bitwarden is breached/decrypted, the attacker gets valid credentials for low-value sites (which you can rotate easily) but useless dummy data for your critical life accounts. They cannot access your email or bank even with the decrypted database. 
Strategy B: The "Keyfile" Approach (Medium Paranoia)

Store your Bitwarden master password as usual.
Enable Two-Step Login on Bitwarden using only your OnlyKey (FIDO2/WebAuthn).
Crucial Step: Do not store the Bitwarden recovery codes in Bitwarden.  Store them on the OnlyKey or printed on paper in a safe.
Result: An attacker who decrypts your data (somehow) still cannot log into your Bitwarden account from a new device without the physical OnlyKey. The "cloud breach" gives them the encrypted blob, which is useless without the master password (which they presumably have in this scenario) AND the physical second factor if you enforce it for login. Note: If you assume the master password is ALSO compromised, this fails, which leads back to Strategy A. 



Bitwarden split knowledge password strategy

View all
Recommendation
Do not manually append memory words to every password. The risk of lockout and the "single keystroke capture" vulnerability outweighs the theoretical benefit. 

Implement Strategy A (Split Storage):

Keep Bitwarden for 95% of sites (convenience + cloud backup).
Move your top 20 critical passwords to OnlyKey slots.
Replace the passwords in Bitwarden for those 20 sites with random gibberish that you never use.
If Bitwarden is fully compromised, your "crown jewels" remain physically isolated on your hardware key. 
This gives you the resiliency of the cloud for the bulk of your data, but ensures that a total cloud compromise does not result in total identity theft.


Copy


should I abandon keepassxc then?


Finished
No, do not abandon KeePassXC entirely, but stop using it as your sole password store. 

Your ZFS corruption scare proved that local-only storage is a single point of failure. However, KeePassXC remains superior for offline isolation and Qubes OS integration. The solution is not to delete it, but to change how you use it within a Hybrid Defense Strategy.

The "Tiered Resiliency" Strategy
Instead of choosing one or the other, assign specific roles to each tool based on your threat model:

1. The "Cloud Resilience" Layer (Bitwarden)
Role: Primary store for 95% of accounts (forums, shopping, low-risk services).
Why: Solves the ZFS/corruption risk instantly via cloud versioning and redundancy.
Security Mitigation: Since you fear a cloud breach, do not store TOTP secrets here.  Use your OnlyKey for 2FA. If Bitwarden is breached, attackers get passwords but cannot generate the 2FA codes to use them.
2. The "High-Value Vault" (KeePassXC in Qubes)
Role: Store only your top 20 critical accounts (Banking, Primary Email, Tax, Medical) and system recovery codes. 
Why: Keeps your "crown jewels" off the cloud entirely. Even if Bitwarden is decrypted by an attacker, these accounts remain inaccessible.
Fixing the Corruption Risk: You must implement automated, encrypted offsite backups for this specific database.
Action: Script your Qubes Vault to export the .kdbx file to a networked VM, encrypt it with a secondary passphrase (or GPG), and sync it to a cold storage cloud bucket (e.g., Proton Drive or AWS S3) or a physical USB drive rotated weekly.
Result: You retain the isolation of Qubes but eliminate the "disk corruption = total loss" scenario. 

Qubes OS automated encrypted backup script Vault

View all
Why "Split Knowledge" on Every Password Fails
You previously asked about appending a "memory word" to every password. Do not do this.

Keyloggers: If your device is compromised, a keylogger captures your typed prefix/suffix immediately, rendering the split useless.
Usability: Manual entry on mobile devices leads to typos and lockouts.
Better Split: Split the storage, not the string. Keep the password in KeePassXC (offline) and the TOTP code on OnlyKey (hardware).  This forces an attacker to breach your cloud, hack your offline vault, and steal your physical key to compromise an account.
Final Recommendation
Keep KeePassXC: It is essential for your high-security Qubes workflow. 
Fix Backups: Implement the 3-2-1 rule (3 copies, 2 media types, 1 offsite) for your KeePass database immediately to prevent future ZFS panic.
Adopt Bitwarden for Bulk: Move low-risk passwords to Bitwarden for convenience and cloud safety. 
Hardware Anchor: Use OnlyKey for all TOTP and FIDO2, ensuring that neither password manager holds the "keys to the kingdom" alone.
This approach gives you the best of both worlds: the disaster recovery of the cloud and the fortress-like isolation of Qubes.


Copy


I like the above. I guess I'm abandoning my yubikeys. I own about 4-5 of them... Maybe give them to close family?



Finished
Do not abandon them yet. While OnlyKey is superior for your specific "split-knowledge" and backup workflow, YubiKey remains the gold standard for pure FIDO2 authentication and mobile NFC support (which OnlyKey lacks). 

Your 4-5 YubiKey devices are valuable assets, but their role in your strategy should shift from "primary vault" to "specialized authenticators." 

Strategic Repurposing Plan
1. The "Family Trust" Strategy (Highly Recommended) Giving them to close family is an excellent idea, provided you manage the setup:

Action: Gift 1-2 keys to trusted family members (spouse, parents, adult children).
Setup: Help them register the keys on their critical accounts (Email, Banking, Apple/Google ID).
Benefit: This secures your "inner circle" against phishing, which is the #1 cause of account takeovers. It also creates a recovery alliance: if you are ever locked out of your own accounts, a trusted family member with a recovery code (that you gave them physically) can help you regain access, and vice versa.
Caveat: Ensure they understand not to lose it. Unlike your OnlyKey setup, if they lose the YubiKey and didn't save recovery codes, their accounts are gone. 
2. The "Air-Gapped" Cold Storage Keep 1-2 YubiKey devices as emergency cold backups for your most critical accounts (e.g., Bitwarden Master Account, Primary Email).

Why: Since YubiKey firmware is immutable and uses a secure element, it is theoretically more resistant to physical extraction attacks than OnlyKey.
Method: Register these keys on your top 5 accounts, then store them in a fireproof safe or off-site location (e.g., a safe deposit box). They are your "break glass in case of emergency" keys if your OnlyKey devices are ever lost, stolen, or compromised simultaneously.
3. The "Legacy Protocol" Specialist Use one YubiKey for specific protocols where it outperforms OnlyKey:

NFC/Mobile: If you ever need to authenticate on an Android phone or iPhone, YubiKey's NFC models are far more convenient than OnlyKey's USB-only form factor. 
PIV/Smart Card: If you ever need to log into a corporate Windows domain or use a government site requiring a PIV/CAC card, YubiKey supports this natively; OnlyKey does not.
Critical Warning: The "EUCLEAK" Vulnerability
Before gifting or storing them, check the firmware version on each YubiKey. 

The Issue: A 2024 vulnerability dubbed EUCLEAK affects YubiKey 5 Series devices with firmware older than 5.7.  This flaw allows a sophisticated attacker with physical access to clone the key.
The Fix: If a key is on firmware < 5.7, it cannot be patched (YubiKeys are immutable).  You must either:
Contact Yubico for a replacement (they offered a program for this).
Do not gift these specific units for high-security use.  Use them only for low-risk accounts or non-critical family members, and advise them of the limitation.
Keys with firmware 5.7 or newer are safe. 

YubiKey EUCLEAK vulnerability check firmware version

View all
Final Verdict
Keep 2 as cold storage, gift the rest (if firmware > 5.7). Your OnlyKey duo will handle your daily "split-knowledge" workflow and password management perfectly. The YubiKeys are now your disaster recovery insurance and a way to upgrade your family's security posture. Just ensure you verify the firmware version first to avoid gifting a vulnerable device. 

