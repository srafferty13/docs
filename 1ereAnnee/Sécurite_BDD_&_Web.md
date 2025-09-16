## Revision B3.4 (Sécurité BDD & Web)

### 1. Gestion des droits d’accès (B3.4.1)
#### MySQL
```sql
CREATE USER 'internaute'@'localhost' IDENTIFIED BY 'MotDePasseComplexe123!';
GRANT SELECT ON Dutoit.* TO 'internaute'@'localhost';
FLUSH PRIVILEGES;
REVOKE UPDATE ON Dutoit.utilisateur FROM 'assistant'@'localhost';
DROP USER 'internaute'@'localhost';
```

#### PostgreSQL
```sql
CREATE ROLE assistant WITH LOGIN PASSWORD 'MotDePasseComplexe123!';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO assistant;
```

#### Bonnes pratiques
- Ne jamais utiliser "root" pour une app web.
- Supprimer les comptes par défaut.
- Vérifier les droits : `SHOW GRANTS FOR 'assistant'@'localhost';`

### 2. Chiffrement et protection des données (B3.4.2)
#### Hachage PHP (bcrypt)
```php
$hash = password_hash("MotDePasse123!", PASSWORD_DEFAULT);
$verif = password_verify("MotDePasse123!", $hash);
```

#### Libsodium (chiffrement symétrique)
```php
$cle = random_bytes(SODIUM_CRYPTO_SECRETBOX_KEYBYTES);
$nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
$chiffre = sodium_crypto_secretbox("Donnée secrète", $nonce, $cle);
```

#### Pseudonymisation vs Anonymisation
- **Pseudonymisation** : réversible (ex: chiffrer nom/prénom).
- **Anonymisation** : irréversible (ex: supprimer email/téléphone).

### 3. Respect du RGPD (B3.4.3)
- Droits des utilisateurs : information, accès, rectification, portabilité, oubli.
- Consentement : cases décochées par défaut, bannière cookies obligatoire.
- Portabilité (API REST) :
  ```php
  header('Content-Type: application/json');
  echo json_encode(["nom" => "Dupont", "email" => "dupont@exemple.com"]);
  ```

### 4. Prévention des injections SQL (B3.4.4)
#### Requêtes préparées (PDO)
```php
$stmt = $db->prepare("SELECT * FROM users WHERE email = :email");
$stmt->bindValue(':email', $email, PDO::PARAM_STR);
$stmt->execute();
```

#### Validation des saisies
- `filter_var()`, `is_numeric()`
- `htmlspecialchars()` pour protéger les entrées utilisateur.

#### Types d’attaques
- Commentaires : `'admin'--'`
- UNION : `SELECT * FROM produits UNION SELECT * FROM users`
- OR 1=1 : `SELECT * FROM users WHERE user='admin' OR 1=1`

### 5. Sécurité des sessions (B3.4.5)
#### Démarrage et sécurisation
```php
session_start();
$_SESSION['user_id'] = 123;
session_set_cookie_params([
    'lifetime' => 3600,
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict'
]);
```

#### Prévention fixation de session
```php
session_regenerate_id(true);
```

#### CSRF
- Utiliser un jeton (token) unique dans les formulaires.

#### Jetons OTP
- Mot de passe à usage unique (ex: SMS, Google Authenticator).

### 6. Bonnes pratiques générales
- ✅ HTTPS obligatoire
- ✅ Mots de passe forts : 12+ caractères, majuscules, chiffres, symboles
- ✅ Journalisation des accès
- ✅ Mises à jour régulières (PHP, OpenSSL, etc.)

---
