---
title: "Too late to understand enums in database?"
date: 2024-12-12
tags: ["database"]
draft: false
---

So today in one of the discussion, which is related to database schema, my TL mentioned that "do not use enums here". I agreed and said yes. I will not do it; this is just for demonstration purpose.
Later in the day I got a flashback when my previous TL said the same, that do not use enums in database and use string instead.

I never gave it a thought before. I used to just go ahead with instructions, but now I wanted to know why. So, I chatted with LLM to find out the truth behind this practice.

I learned that an enum is a datatype defined at the database level, and it is not only a string. They are named values called elements, members.
This was new to me. I never paid attention to this in 9 years of my programming journey. But it's never too late.

An enum is a type in database that allows us to restrict a column to accept only a predefined set of values.

## So why do some developers avoid them?

- If we think that value of a column may change frequently then it is not recommended to use enum, because to alter values of enum, we need to do database migration.
It is easier to handle this in application logic.

- They say enums are implemented differently across databases. So if we switch our ORM or database then it will be a overhead for us.

## When to use enum?

- If we know that a field's values are never going to change, like status `active`, `inactive` then it is recommended to use enum. Because they say, enum might offer better performance for certain queries due to their internal representation.

- If we want to ensure that invalid values must not be stored in database then enum can enforce this for us. So invalid or misspelled entries will get rejected by database itself.

- Manual queries or scripts can insert invalid data if not carefully managed. So enum will help us avoid that.

- Enums can be more storage-efficient than string, especially for large datsets.

## Conclusion

We are told to avoid enums because we want flexibility, ease of maintainance and simplicity in deployment process.
Enums do provide us strong data integrity at database level but they can introduce complexity and rigidity that may not be desirable in current project.


---

## Writing Improvement Tips For Self

- Ensure Subject-Verb Agreement. Make sure that verbs agree in number with their subjects.
  **Example:** Use "developers **avoid**" instead of "developers **avoids**".

- Include Articles Before Nouns
 **Example:** Write "I got **a** flashback" instead of "I got flashback".

- Use proper punctuation to separate independent clauses.

- Use Correct Word Forms and Verb Tenses
  **Example:** Use "I had never **paid** attention" instead of "I never **gave** attention".
