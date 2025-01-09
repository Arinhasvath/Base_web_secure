# Base_web_secure

### Structure de fichiers

```
jwt_auth_site/
│
├── frontend/
│   ├── index.html
│   ├── login.html
│   ├── register.html
│   ├── style.css
│   └── script.js
│
└── backend/
    ├── app.py
    ├── requirements.txt
    └── models.py
```

### 1. Fichiers du frontend

#### `frontend/index.html`
```html
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
```

#### `frontend/login.html`
```html
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
```

#### `frontend/register.html`
```html
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
```

#### `frontend/style.css`
```css
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
```

#### `frontend/script.js`
```javascript
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
```

### 2. Fichiers du backend

#### `backend/app.py`
```python
from flask import Flask, jsonify, request
from flask_jwt_extended import JWTManager, create_access_token, jwt_required
from models import User, users_db

app = Flask(__name__)
app.config['JWT_SECRET_KEY'] = 'your_jwt_secret_key'  # Change it to a random secret key
jwt = JWTManager(app)

@app.route('/api/register', methods=['POST'])
def register():
    data = request.json
    username = data.get('username')
    email = data.get('email')
    password = data.get('password')

    if not username or not email or not password:
        return jsonify({'message': 'Tous les champs sont obligatoires.'}), 400

    # Check if user already exists
    for user in users_db:
        if user['email'] == email:
            return jsonify({'message': 'Utilisateur déjà existant.'}), 400

    # Create new user
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

@app.route('/api/protected', methods=['GET'])
@jwt_required()
def protected():
    return jsonify({'message': 'Ceci est une route protégée.'}), 200

if __name__ == '__main__':
    app.run(debug=True)
```

#### `backend/models.py`
```python
# Base de données fictive en mémoire
users_db = []
```

#### `backend/requirements.txt`
```
Flask
Flask-JWT-Extended
```

### Instructions

1. **Configuration de l'environnement backend :**
   - Crée un environnement virtuel (optionnel, mais recommandé) :
     ```bash
     python -m venv venv
     source venv/bin/activate  # Sur Windows : venv\Scripts\activate
     ```
   - Installe les dépendances :
     ```bash
     pip install -r requirements.txt
     ```

2. **Lancer le backend :**
   - Dans le dossier `backend`, exécute :
     ```bash
     python app.py
     ```
   - Cela démarrera le serveur Flask sur `http://localhost:5000`.

3. **Lancer le frontend :**
   - Ouvre les fichiers `frontend/index.html`, `frontend/login.html` et `frontend/register.html` dans un navigateur pour afficher les pages.

### Résumé
Cette structure de base te permet de démarrer un projet avec un système d'inscription et de connexion sécurisé par JWT. Tu peux développer et ajouter des fonctionnalités supplémentaires au fur et à mesure de tes besoins. Assure-toi de gérer la sécurité des mots de passe en utilisant des techniques de hachage dans une application de production.

