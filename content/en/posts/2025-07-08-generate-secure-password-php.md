---
title: 'Generate a Secure Password in PHP (2025 Version)'
date: '2025-07-08T10:00:00+02:00'
draft: false
translationKey: "generer-mot-de-passe-securise-php"
---

Generating strong passwords is a fundamental element of application security. A strong password should be long and contain a combination of different character types.

An outdated approach, often seen in old tutorials, used functions like `rand()` and `srand()`. These are **not** considered secure enough for cryptographic tasks. Here is a modern and secure solution for generating passwords in PHP.

### The Modern Function: `generateStrongPassword`

This new function uses `random_int()`, which is designed in PHP to generate cryptographically secure random integers. It is also much more flexible and readable.

```php
/**
 * Generates a secure password.
 *
 * @param int  $length Desired password length.
 * @param bool $includeDigits Include digits.
 * @param bool $includeUppercase Include uppercase letters.
 * @param bool $includeLowercase Include lowercase letters.
 * @param bool $includeSymbols Include symbols.
 * @return string The generated password.
 * @throws \Exception If secure random number generation fails.
 */
function generateStrongPassword(
    int $length = 12,
    bool $includeDigits = true,
    bool $includeUppercase = true,
    bool $includeLowercase = true,
    bool $includeSymbols = true
): string {
    $characterSets = [];
    if ($includeDigits) {
        $characterSets[] = '0123456789';
    }
    if ($includeUppercase) {
        $characterSets[] = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    }
    if ($includeLowercase) {
        $characterSets[] = 'abcdefghijklmnopqrstuvwxyz';
    }
    if ($includeSymbols) {
        // You can customize this list of symbols
        $characterSets[] = '!@#$%^&*()_+-=[]{}|';
    }

    if (empty($characterSets)) {
        throw new \InvalidArgumentException('At least one character set must be selected.');
    }

    $password = '';
    $allCharacters = implode('', $characterSets);

    // Guarantees at least one character from each requested set for robustness
    foreach ($characterSets as $set) {
        $password .= $set[random_int(0, strlen($set) - 1)];
    }

    // Completes the rest of the password
    $remainingLength = $length - count($characterSets);
    if ($remainingLength > 0) {
        for ($i = 0; $i < $remainingLength; $i++) {
            $password .= $allCharacters[random_int(0, strlen($allCharacters) - 1)];
        }
    }

    // Shuffles the password so that the first characters are not predictable
    return str_shuffle($password);
}
```

### Why is this approach better?

1.  **Increased Security**: `random_int()` is designed for cryptographic needs, unlike `rand()` which produces predictable numbers and should never be used for security.
2.  **Flexibility**: Instead of an unintuitive `$strength` parameter, the new function uses explicit booleans (`$includeDigits`, `$includeUppercase`, etc.). This makes the code more readable and easier to use.
3.  **Guaranteed Robustness**: The function ensures that if you request digits, uppercase letters, and symbols, the password will contain *at least one* character of each requested type, before completing the rest.
4.  **Modern Code**: The use of strict typing (`int`, `bool`, `string`) and exceptions makes the function more robust and integrable into modern projects.

### Code Explanation

1.  **`$characterSets`**: An array that will contain the possible character strings (digits, uppercase, etc.) depending on the parameters.
2.  **`random_int(min, max)`**: The cornerstone of our function. It generates a high-quality random number between `min` and `max`.
3.  **Character Set Guarantee**: The first `foreach` loop iterates through the requested character sets and adds a random character from each set to the password. This prevents having a password that, by bad luck, would only contain lowercase letters when you had requested others.
4.  **Padding**: The next `for` loop completes the password to the desired length by randomly drawing from the set of all allowed characters.
5.  **`str_shuffle()`**: This last step is crucial. It shuffles the final character string to ensure that the first characters (those guaranteed by the first loop) do not follow a predictable order (e.g., always a digit, then an uppercase letter, etc.).

### Usage Examples

```php
// 16-character password with all character types (default)
echo generateStrongPassword(16);
// Possible result: j!6P@sWqK_zR{gE8

// Simple 8-character password, with only lowercase letters and digits
echo generateStrongPassword(8, true, false, true, false);
// Possible result: 4p7k1n9z

// 10-character password without symbols
echo generateStrongPassword(10, true, true, true, false);
// Possible result: 9aB4rT2zK1
```

By adopting this modern approach, you significantly improve the security of password generation in your PHP applications.
