---
title: "Improper Error Handling Pentesting Checklist"
layout: default
---

# Improper Error Handling Pentesting Checklist

## Error Response Discovery

1. Identify all application endpoints and parameters.
2. Send malformed requests to normal endpoints.
3. Remove required parameters.
4. Add unexpected parameters.
5. Send empty parameter values.
6. Send null values where applicable.
7. Send excessively long parameter values.
8. Send unexpected data types.
9. Send arrays where strings are expected.
10. Send objects where primitive values are expected.
11. Send negative numbers.
12. Send extremely large numbers.
13. Send floating-point values where integers are expected.
14. Send invalid Boolean representations.
15. Send malformed JSON.
16. Send malformed XML.
17. Send malformed form-data.
18. Send incomplete multipart requests.
19. Send invalid URL/URI values.
20. Send invalid date/time formats.

---

## HTTP-Level Exception Handling

1. Test unsupported HTTP methods.
2. Test unexpected HTTP methods on authenticated endpoints.
3. Test malformed HTTP headers.
4. Remove required headers.
5. Duplicate security-sensitive headers.
6. Send invalid Content-Type.
7. Send mismatched Content-Type and request body.
8. Send invalid Accept headers.
9. Test malformed Authorization headers.
10. Test expired authentication tokens.
11. Test structurally invalid tokens.
12. Test requests with missing cookies.
13. Test malformed cookies.
14. Test oversized cookies.
15. Test invalid Origin values.
16. Test malformed Referer values.
17. Test invalid Host handling where authorized.
18. Test unsupported content encodings.
19. Test malformed chunked requests where applicable.
20. Check whether errors consistently return appropriate HTTP status codes.

---

## Information Disclosure Through Errors

1. Search responses for stack traces.
2. Search for exception class names.
3. Search for framework names and versions.
4. Search for library names and versions.
5. Look for source-code snippets.
6. Look for source-code file paths.
7. Look for internal class names.
8. Look for method/function names.
9. Look for database table names.
10. Look for database column names.
11. Look for SQL queries.
12. Look for filesystem paths.
13. Look for internal IP addresses.
14. Look for internal hostnames.
15. Look for usernames/service accounts.
16. Look for environment variables.
17. Look for configuration values.
18. Look for cloud resource identifiers.
19. Look for debugging information.
20. Look for request IDs that expose internal implementation details.

---

## Database Exception Handling

1. Submit invalid database-related input.
2. Test type mismatches that reach database operations.
3. Test invalid identifiers.
4. Test invalid sorting/filtering parameters.
5. Test invalid pagination values.
6. Test duplicate values where uniqueness is expected.
7. Trigger constraint violations.
8. Test malformed database query parameters.
9. Check for SQL/database error messages.
10. Identify database engine information.
11. Check whether failed transactions expose sensitive information.
12. Check whether database exceptions return different responses for different users.
