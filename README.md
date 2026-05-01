# Stored XSS Lab

## Description
This lab demonstrates a stored XSS vulnerability.

## Vulnerability
User input is stored and later rendered using innerHTML.

## Impact
Malicious code is executed for every user.

## Payload
<img src=x onerror="alert('Stored XSS')">

## Fix
Use textContent instead of innerHTML.

## Conclusion
Never store and render user input without sanitization.