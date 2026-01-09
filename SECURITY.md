# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |

## Reporting a Vulnerability

We take security issues seriously. If you discover a security vulnerability in pandas-airtable, please report it responsibly.

### How to Report

1. **Do NOT** open a public GitHub issue for security vulnerabilities
2. Email the maintainers directly at dhunganasujal9@gmail.com
3. Include as much detail as possible:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

### What to Expect

- **Acknowledgment**: We will acknowledge receipt of your report within 48 hours
- **Assessment**: We will assess the vulnerability and determine its severity
- **Updates**: We will keep you informed of our progress
- **Resolution**: We aim to resolve critical vulnerabilities within 7 days
- **Credit**: We will credit you in the release notes (unless you prefer to remain anonymous)

## Security Best Practices

When using pandas-airtable, follow these security best practices:

### API Key Management

1. **Never commit API keys** to version control
2. Use environment variables or secret management tools:
   ```python
   import os
   api_key = os.environ["AIRTABLE_API_KEY"]
   ```
3. Use the minimum required permissions for your API key
4. Rotate API keys periodically

### Data Handling

1. Be cautious when reading data from untrusted Airtable bases
2. Validate and sanitize data before processing
3. Don't expose sensitive data in logs or error messages

### Dependencies

1. Keep pandas-airtable and its dependencies up to date
2. Regularly check for security advisories
3. Use tools like `pip-audit` or `safety` to scan for vulnerabilities

## Security Features

pandas-airtable includes several security-conscious features:

- **No credential storage**: API keys are never stored or cached
- **Rate limiting**: Built-in rate limiting prevents accidental API abuse
- **Input validation**: Parameters are validated before API calls
- **Error handling**: Sensitive information is not exposed in error messages

## Disclosure Policy

We follow a coordinated disclosure policy:

1. Reporter contacts us with vulnerability details
2. We confirm the vulnerability and assess impact
3. We develop and test a fix
4. We release the fix and publish a security advisory
5. We publicly credit the reporter (if desired)

Thank you for helping keep pandas-airtable secure!
