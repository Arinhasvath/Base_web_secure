# Script pour créer un projet web avec authentification JWT

## Introduction

### Script à copier dans le terminal

Assure-toi d'être dans le répertoire où tu souhaites créer ton projet, puis exécute le script suivant :

```bash
# Créer la structure de dossiers et fichiers
mkdir -p jwt_auth_site/frontend
mkdir -p jwt_auth_site/backend

# Fichiers du frontend
cat <<EOL > jwt_auth_site/frontend/index.html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accueil</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <img src="logo.png" alt="Logo" class="logo">
        <h1 class="title">Mon Site Web</h1>
        <div class="auth-links">
            <a href="login.html">Connexion</a>
            <a href="register.html">Inscription</a>
        </div>
    </header>
    <main>
        <h2>Bienvenue sur notre site !</h2>
    </main>
</body>
</html>
EOL

cat <<EOL > jwt_auth_site/frontend/login.html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Connexion</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <img src="logo.png" alt="Logo" class="logo">
        <h1 class="title">Connexion</h1>
    </header>
    <main>
        <form id="loginForm">
            <input type="email" placeholder="Email" required>
            <input type="password" placeholder="Mot de passe" required>
            <button type="submit">Se connecter</button>
        </form>
    </main>
    <script src="script.js"></script>
</body>
</html>
EOL

cat <<EOL > jwt_auth_site/frontend/register.html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inscription</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <img src="logo.png" alt="Logo" class="logo">
        <h1 class="title">Inscription</h1>
    </header>
    <main>
        <form id="registerForm">
            <input type="text" placeholder="Nom d'utilisateur" required>
            <input type="email" placeholder="Email" required>
            <input type="password" placeholder="Mot de passe" required>
            <button type="submit">S'inscrire</button>
        </form>
    </main>
    <script src="script.js"></script>
</body>
</html>
EOL

cat <<EOL > jwt_auth_site/frontend/style.css
body {
    font-family: Arial, sans-serif;
    background-color: #f2f2f2;
    margin: 0;
}

header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #00698f;
    color: white;
    padding: 10px 20px;
}

.logo {
    height: 50px;
}

.title {
    flex-grow: 1;
    text-align: center;
}

.auth-links a {
    color: white;
    margin: 0 10px;
    text-decoration: none;
}

main {
    text-align: center;
    padding: 20px;
}

form {
    display: flex;
    flex-direction: column;
    align-items: center;
}

input {
    margin: 10px 0;
    padding: 10px;
    width: 200px;
}

button {
    padding: 10px 20px;
    background-color: #00698f;
    color: white;
    border: none;
    cursor: pointer;
}
EOL

cat <<EOL > jwt_auth_site/frontend/script.js
document.getElementById('loginForm')?.addEventListener('submit', function(event) {
    event.preventDefault();
    const email = this[0].value;
    const password = this[1].value;

    fetch('http://localhost:5000/api/login', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, password })
    })
    .then(response => response.json())
    .then(data => {
        if (data.token) {
            localStorage.setItem('token', data.token);
            alert('Connexion réussie !');
            window.location.href = 'index.html';
        } else {
            alert('Erreur de connexion : ' + data.message);
        }
    })
    .catch(error => console.error('Erreur:', error));
});

document.getElementById('registerForm')?.addEventListener('submit', function(event) {
    event.preventDefault();
    const username = this[0].value;
    const email = this[1].value;
    const password = this[2].value;

    fetch('http://localhost:5000/api/register', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ username, email, password })
    })
    .then(response => response.json())
    .then(data => {
        if (data.message) {
            alert(data.message);
            window.location.href = 'login.html';
        } else {
            alert('Erreur d\'inscription : ' + data.message);
        }
    })
    .catch(error => console.error('Erreur:', error));
});
EOL

# Fichiers du backend
cat <<EOL > jwt_auth_site/backend/app.py
from flask import Flask, jsonify, request
from flask_jwt_extended import JWTManager, create_access_token
from models import users_db

app = Flask(__name__)
app.config['JWT_SECRET_KEY'] = 'your_jwt_secret_key'  # Change it to a random secret key
jwt = JWTManager(app)

@app.route('/api/register', methods=['POST'])
def register():
    data = request.json
    username = data.get('username')
    email = data.get('email')
    password = data.get('password')

    # Vérifier si l'utilisateur existe déjà
    for user in users_db:
        if user['email'] == email:
            return jsonify({'message': 'Utilisateur déjà existant.'}), 400

    # Créer un nouvel utilisateur
    new_user = {'username': username, 'email': email, 'password': password}
    users_db.append(new_user)
    return jsonify({'message': 'Inscription réussie !'}), 201

@app.route('/api/login', methods=['POST'])
def login():
    data = request.json
    email = data.get('email')
    password = data.get('password')

    user = next((user for user in users_db if user['email'] == email), None)
    if user and user['password'] == password:
        access_token = create_access_token(identity={'username': user['username'], 'email': user['email']})
        return jsonify({'token': access_token}), 200
    return jsonify({'message': 'Email ou mot de passe incorrect.'}), 401

if __name__ == '__main__':
    app.run(debug=True)
EOL

cat <<EOL > jwt_auth_site/backend/models.py
# Base de données fictive en mémoire
users_db = []
EOL

cat <<EOL > jwt_auth_site/backend/requirements.txt
Flask
Flask-JWT-Extended
EOL

cat <<EOL > jwt_auth_site/README.md
# Projet de Site Web avec Authentification JWT

## Description
Ce projet est un site web simple avec un système d'inscription et de connexion utilisant JWT pour l'authentification.

## Installation
1. Clonez le dépôt ou téléchargez les fichiers.
2. Naviguez dans le dossier `backend` et installez les dépendances :
   \`\`\`bash
   pip install -r requirements.txt
   \`\`\`

3. Lancez le serveur :
   \`\`\`bash
   python app.py
   \`\`\`

4. Ouvrez les fichiers HTML dans un navigateur pour utiliser l'application.

## Technologies
- Flask
- Flask-JWT-Extended
- HTML/CSS/JavaScript
EOL

# Afficher un message de confirmation
echo "La structure du projet a été créée avec succès !"
```

### Instructions pour exécuter le script

1. **Ouvre ton terminal**.
2. **Navigue jusqu'au répertoire** où tu souhaites créer ton projet.
3. **Colle le script** dans ton terminal et appuie sur `Entrée`.

### Ce que fait le script

- Il crée une structure de dossiers pour le frontend et le backend.
- Il crée les fichiers HTML pour l'interface utilisateur, le fichier CSS pour le style, le fichier JavaScript pour la logique, et les fichiers Python pour le backend.
- Il crée également un fichier `requirements.txt` pour les dépendances Python et un fichier `README.md` pour la documentation du projet.