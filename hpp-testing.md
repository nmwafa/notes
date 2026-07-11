---
title: "HPP Testing Checklist"
layout: default
---

# HTTP Parameter Pollution (HPP) Testing Checklist

## 1. Identify Input Parameters
- Identify all GET parameters.
- Identify all POST parameters.
- Identify JSON body parameters.
- Identify XML parameters.
- Identify multipart/form-data parameters.
- Identify GraphQL variables.
- Identify hidden form parameters.
- Identify URL path parameters converted into query parameters.
- Identify REST API query parameters.
- Identify authentication-related parameters.
- Identify authorization parameters.
- Identify callback/redirect parameters.
- Identify search/filter/sort parameters.
- Identify pagination parameters.
- Identify file upload parameters.

## 2. Basic Parameter Duplication
- Duplicate GET Parameter: ?id=1&id;=2
- Check whether first value is used.
- Check whether last value is used.
- Check whether values are concatenated.
- Check whether both values are processed.
- Check for application errors.
- Duplicate POST Parameter: username=admin & username=user
- Observe which value reaches backend.
- Duplicate JSON Key: {"name":"admin","name":"guest"}
- Check parser behavior.
- Duplicate Cookie: role=user; role=admin
- Check privilege escalation.

## 3. Different Parameter Separators
- Try &, ;, %26, %3B, &&
- Examples: ?id=1&id;=2, ?id=1;id=2, ?id=1%26id=2, ?id=1&&id;=2
- Observe parser differences.

## 4. Empty Parameter Pollution- ?id=1&id;=
- ?id=&id;=1
- ?id=
- Check validation bypass.

## 5. Null Parameter Pollution
- ?id=null&id;=1
- ?id=%00&id;=1
- ?id=undefined&id;=1

## 6. Array Style Pollution
- id[]=1&id;[]=2
- id[0]=1&id;[1]=2
- id[]=1&id;=2
- id=1&id;[]=2
- Observe backend parsing.
