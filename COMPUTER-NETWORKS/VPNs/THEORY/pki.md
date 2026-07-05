# 🔐 PKI & Cryptography: The Story of Jarek, Marek, and the CA

To truly understand how Public Key Infrastructure (PKI), Certificates, and Digital Signatures work, let's break it down into a simple story involving two users (Jarek and Marek) and a trusted third party (the Certificate Authority - CA).


### 📖 The Story (Step-by-Step)

#### Phase 1: Preparation (At Jarek's House)
1.  Jarek wants to set up a secure server. He generates a Key Pair locally: a **Private Key** (which he hides deep on his hard drive) and a **Public Key**.
2.  Jarek creates a **CSR (Certificate Signing Request)** file. He puts his name, his domain's IP address, and his Public Key inside it.
3.  Jarek sends this CSR file to a Certificate Authority (CA) and pays them a fee (if it's a public CA like DigiCert).

#### Phase 2: The CA's Job (Issuing the Certificate)
1.  The CA verifies that Jarek is actually Jarek (e.g., by making him click a link sent to the domain's admin email).
2.  The CA takes Jarek's data from the CSR and creates a plain-text document from it—the **Certificate**.
3.  **CREATING THE SIGNATURE (The most important part!):** The CA calculates a hash of Jarek's certificate. Then, the CA takes its own, super-secret **CA Private Key** and "encrypts" this hash with it. This encrypted hash is the **Digital Signature**.
4.  The CA attaches this Signature to the very bottom of Jarek's certificate and sends the finished file back to him.

#### Phase 3: Verification (Marek connects to Jarek)
1.  Marek visits Jarek's website. Jarek sends Marek his Certificate (the plain text + the CA's signature at the bottom).
2.  Marek must check if this certificate is forged.
3.  Marek has the **Root CA Certificate** (which contains the **CA's Public Key**) built directly into his Windows operating system.
4.  Marek takes this CA Public Key from his system and uses it to decrypt the signature on Jarek's certificate.
5.  *The Magic:* If the signature can be decrypted using the CA's Public Key, Marek has 100% mathematical certainty that the certificate MUST have been signed by the CA's Private Key (because only those two keys fit together). Marek compares the hashes. They match! Jarek's identity is confirmed.

#### Phase 4: Secure Communication (Encryption)
1.  Since Marek now trusts Jarek, he extracts **Jarek's Public Key** from the certificate.
2.  Marek encrypts a secret message (e.g., his credit card number) using Jarek's Public Key and sends it over the internet.
3.  Even if a hacker intercepts it, they cannot read it. ONLY Jarek, who possesses **Jarek's Private Key**, can decrypt this message.

---

> **🏆 The Golden Rule of PKI (Memorize this for the exam!):**
> 
> *   You want to **ENCRYPT** data for someone? ➡️ You use **THEIR Public Key**.
> *   You want to **SIGN** something (prove it's you)? ➡️ You use **YOUR Private Key**.
> *   You want to **VERIFY** someone's signature? ➡️ You use **THEIR Public Key**.


Polski

historia Marka i Jarka (Krok po kroku na GitHuba)

Faza 1: Przygotowania (U Jarka w domu)


1. Jarek chce postawić bezpieczny serwer. Generuje u siebie parę kluczy: Prywatny (chowa głęboko na dysku) i Publiczny.
2. Jarek tworzy plik CSR (Certificate Signing Request). Wrzuca do niego swoje imię, adres IP domeny i swój Klucz Publiczny.
3. Jarek wysyła ten plik CSR do Urzędu CA (i płaci im kasę, jeśli to publiczne CA, np. DigiCert).

Faza 2: Praca Urzędu CA (Wydanie certyfikatu)


1. Urząd CA weryfikuje, czy Jarek to naprawdę Jarek (np. każe mu kliknąć w link wysłany na maila domeny).
2. Urząd CA bierze dane Jarka z CSR i tworzy z nich jawny dokument tekstowy – Certyfikat.
3. TWORZENIE PODPISU (Najważniejsze!): Urząd CA wylicza hash (skrót) z tego certyfikatu Jarka. Następnie CA bierze swój własny, super-tajny Klucz Prywatny CA i "szyfruje" nim ten hash. Ten zaszyfrowany hash to właśnie Podpis Cyfrowy (Digital Signature).
4. CA dokleja ten Podpis na sam dół certyfikatu Jarka i odsyła mu gotowy plik.

Faza 3: Weryfikacja (Marek łączy się z Jarkiem)


1. Marek wchodzi na stronę Jarka. Jarek wysyła Markowi swój Certyfikat (jawny tekst + podpis CA na dole).
2. Marek musi sprawdzić, czy ten certyfikat nie jest podrobiony.
3. Tak jak świetnie napisałeś: Marek ma w swoim Windowsie wbudowany Certyfikat Root CA (który zawiera Klucz Publiczny CA).
4. Marek bierze ten Klucz Publiczny CA ze swojego systemu i używa go do odszyfrowania podpisu na certyfikacie Jarka.
5. Jeśli podpis da się odszyfrować kluczem publicznym CA, to Marek ma 100% matematycznej pewności, że ten certyfikat musiał zostać podpisany Kluczem Prywatnym CA (bo tylko te dwa klucze do siebie pasują). Marek porównuje hashe. Zgadza się! Tożsamość Jarka jest potwierdzona.

Faza 4: Bezpieczna komunikacja (Szyfrowanie)


1. Skoro Marek już ufa Jarkowi, wyciąga z jego certyfikatu Klucz Publiczny Jarka.
2. Marek szyfruje tajną wiadomość (np. numer karty kredytowej) Kluczem Publicznym Jarka i wysyła w sieć.
3. Nawet jeśli haker to przechwyci, nic nie przeczyta. Tylko Jarek, posiadający swój Klucz Prywatny Jarka, może tę wiadomość odszyfrować.



OPERACJA 1: Tworzenie Certyfikatu (Dzieje się w Urzędzie CA)

Cel: Urząd CA musi stworzyć dokument, którego nikt nie podrobi.


Co mamy na stole: Plik tekstowy, w którym jest napisane: Imię: Jarek, IP: 10.0.0.1, Klucz Publiczny Jarka: XYZ123.
Co robi CA: Algorytm CA wylicza z tego całego tekstu krótki hash (np. SHA-256).
CO DOKŁADNIE SZYFRUJEMY: Urząd CA bierze ten wyliczony HASH i szyfruje go swoim Kluczem Prywatnym CA.
Wynik: Ten zaszyfrowany hash to jest właśnie "Podpis CA". Urząd dokleja ten podpis na sam dół pliku tekstowego. Gotowy plik to Certyfikat.

Wniosek z Operacji 1: Kluczem Prywatnym CA szyfrujemy tylko i wyłącznie hash z danych Jarka, żeby udowodnić, że nikt tych danych nie zmienił po wydaniu certyfikatu.



OPERACJA 2: Nawiązywanie Połączenia (Dzieje się w sieci między Markiem a Jarkiem)

Cel: Jarek musi udowodnić Markowi, że jest prawowitym właścicielem certyfikatu, który właśnie mu wysłał.


Co mamy na stole: Marek odebrał certyfikat Jarka, sprawdził podpis CA i wyciągnął z niego Klucz Publiczny Jarka. Teraz Marek chce mieć pewność, że po drugiej stronie kabla naprawdę siedzi Jarek.
Co robi Marek: Wysyła do Jarka losową, unikalną paczkę danych (tzw. Challenge).
CO DOKŁADNIE SZYFRUJEMY: Jarek bierze tę paczkę danych (lub wylicza z niej hash) i szyfruje ją swoim Kluczem Prywatnym Jarka.
Wynik: Jarek odsyła ten zaszyfrowany kawałek danych do Marka. Marek używa Klucza Publicznego Jarka (z certyfikatu), żeby to odszyfrować. Jeśli się uda – tożsamość Jarka jest w 100% potwierdzona.

Wniosek z Operacji 2: Kluczem Prywatnym Jarka szyfrujemy dane negocjacyjne (Challenge) podczas zestawiania tunelu (np. IPsec/TLS), żeby udowodnić, że Jarek posiada klucz prywatny pasujący do certyfikatu.