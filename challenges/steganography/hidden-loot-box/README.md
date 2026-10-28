# Hidden Loot Box — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Steganography / Web |
| Challenge | Hidden Loot Box |
| Points Available | 15 |
| Challenge Author | @RP-01? |
| Investigation Type | Web Reconnaissance & Hidden Resource Discovery |
| Primary Technique | `robots.txt` Enumeration |
| Current Resource Status | Offline / Inaccessible |
| Documentation Basis | Reconstructed from the original investigation |

> **Investigation Note:** This write-up reconstructs the challenge from my recollection of the original investigation. The challenge website is no longer accessible, preventing independent verification of the full web directory structure and original image. The recovered flag and key investigative steps are preserved below. The original competition submission status is not independently verified in this reconstruction.

---

## 1. Challenge Description & Objective

### Scenario

Never judge a book by its cover. Think you can discover this secret?

### Objective

Explore the provided website, locate concealed information, and recover the hidden flag.

**Required flag format:**

`deadface{flag_text}`

### Provided Resource

```text
http://45.79.200.170:3001/
```

The challenge combined web reconnaissance with information concealed in a resource discovered through the website.

---

## 2. Technical Knowledge & Concepts

### 2.1 Web Reconnaissance

Web reconnaissance involves examining a website's publicly accessible resources and structure to discover potentially relevant information.

Common investigation targets include:

- Page source code
- Linked and unlinked resources
- Publicly accessible configuration files
- Directory paths
- Website metadata
- Search-engine crawler instructions

### 2.2 The robots.txt File

The `robots.txt` file is part of the Robots Exclusion Protocol.

Websites commonly use it to communicate crawling preferences to automated search engines.

It is typically available at the root of a website:

```text
https://example.com/robots.txt
```

A file might contain instructions such as:

```text
User-agent: *
Disallow: /example-directory/
```

This is an illustrative example, **not the recovered contents of the challenge's robots.txt file**.

Although a `Disallow` directive requests that compliant crawlers avoid a path, it does not prevent someone from visiting that path directly.

Therefore, `robots.txt` should never be relied upon to protect confidential resources.

### 2.3 Hidden Resource Discovery

A webpage may contain resources that are not linked from the homepage.

These resources might still be publicly accessible if a visitor knows their URL.

In CTF challenges, `robots.txt` sometimes provides clues to paths containing additional evidence.

### 2.4 Visual Information Hiding

Not all steganography-themed challenges require manipulating image pixels.

Sometimes the intended challenge is locating a concealed image or webpage rather than extracting hidden bits from an image.

In this investigation, the recovered flag was remembered as appearing visibly on an image.

There is no verified evidence that least significant bit extraction or another image-processing algorithm was required.

---

## 3. Investigation Methodology

### Access the Challenge Website

The original investigation began at:

```text
http://45.79.200.170:3001/
```

The website was supplied through the challenge platform's **Click Here** hyperlink.

The challenge description suggested that something important was hidden from immediate view.

### Investigate robots.txt

During the original investigation, I discovered that the website exposed a `robots.txt` file:

```text
http://45.79.200.170:3001/robots.txt
```

This resource provided the next lead in the investigation.

The specific contents of the file were not preserved, so its original directives cannot be reproduced accurately.

### Follow the Discovered Path

Using information obtained from `robots.txt`, I navigated to another webpage.

That page contained an image associated with the challenge.

The exact page URL has not been recovered.

This step demonstrates how an unlinked web resource can still be discovered through publicly accessible website information.

### Examine the Image

The image contained the hidden flag:

```text
deadface{uh_I_feel_exposed}
```

The flag's wording is consistent with the theme of something being exposed through a publicly accessible resource.

The original image is no longer available for inspection in the current investigation.

### Revisit the Website After the Competition

I attempted to access the original website again on October 28, 2026.

The connection failed.

The following command was used:

```bash
curl -I --connect-timeout 5 --max-time 10 \
  http://45.79.200.170:3001/
```

**Observed output:**

```text
curl: (7) Failed to connect to 45.79.200.170 port 3001 after 53 ms: Could not connect to server
```

This established that the challenge endpoint was inaccessible from my investigation environment at that time.

The error does not prove whether the service was permanently shut down or temporarily unavailable.

Because the service was inaccessible, I could not revisit `robots.txt`, recover the exact hidden path, or obtain the original flag image.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Recovered Flag:**

`deadface{uh_I_feel_exposed}`

**Points Available:** 15

**Verification Note:** The flag was recovered from recollection of the original challenge. The original resource is unavailable for independent verification during this documentation session.

</details>

---

## 5. References

- [RFC 9309 — Robots Exclusion Protocol](https://www.rfc-editor.org/rfc/rfc9309)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Google Search Central — Introduction to robots.txt](https://developers.google.com/search/docs/crawling-indexing/robots/intro)

---

[← Return to Steganography Collection](../README.md)
