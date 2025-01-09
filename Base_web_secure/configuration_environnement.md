Voici deux scripts distincts : le premier est pour configurer un environnement de développement pour des applications web avec Flask, et le second est pour créer et configurer une application Flask simple. Chaque script comprend des étapes d’installation et des configurations nécessaires.

### 1. Script pour Configurer l’Environnement de Développement Flask

Ce script va installer Python, pip, un environnement virtuel et Flask.

#### Script Bash pour l’Environnement Flask (`setup_flask_env.sh`)

```bash
#!/bin/bash

# Mettre à jour le système
sudo apt update
sudo apt upgrade -y

# Installer Python et pip
sudo apt install python3 python3-pip python3-venv -y

# Créer un répertoire pour le projet
mkdir -p ~/flask_project
cd ~/flask_project

# Créer un environnement virtuel
python3 -m venv venv

# Activer l'environnement virtuel
# Note: Vous devrez exécuter la commande suivante dans votre terminal
# source venv/bin/activate
echo "Pour activer l'environnement virtuel, exécutez : source venv/bin/activate"

# Installer Flask
source venv/bin/activate
pip install Flask

# Installer d'autres dépendances utiles (optionnel)
pip install Flask-JWT-Extended Flask-CORS

# Créer une structure de projet de base
mkdir -p app/templates app/static

# Créer un fichier README
cat <<EOL > README.md
# Projet Flask

Ce projet est une application Flask de base. Pour démarrer l'environnement virtuel, utilisez :

\`\`\`
source venv/bin/activate
\`\`\`

## Installation des dépendances
Pour installer les dépendances, utilisez :
\`\`\`
pip install -r requirements.txt
\`\`\`
EOL

# Créer un fichier requirements.txt
cat <<EOL > requirements.txt
Flask
Flask-JWT-Extended
Flask-CORS
EOL

echo "L'environnement Flask a été configuré avec succès dans ~/flask_project."
```

### 2. Script pour Créer une Application Flask Simple

Ce script va créer un fichier de base pour une application Flask simple avec une route d'accueil.

#### Script Bash pour l’Application Flask (`create_flask_app.sh`)

```bash
#!/bin/bash

# Assurez-vous d'être dans le bon répertoire
cd ~/flask_project || { echo "Veuillez d'abord exécuter le script setup_flask_env.sh"; exit 1; }

# Créer le fichier principal de l'application
cat <<EOL > app/app.py
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({"message": "Bienvenue sur votre application Flask!"})

if __name__ == '__main__':
    app.run(debug=True)
EOL

# Instructions pour exécuter l'application
echo "L'application Flask a été créée avec succès."
echo "Pour exécuter l'application, suivez ces étapes :"
echo "1. Activez votre environnement virtuel : source venv/bin/activate"
echo "2. Exécutez l'application : python app/app.py"
echo "3. Visitez http://127.0.0.1:5000"
```

### Instructions pour Exécuter les Scripts

1. **Créer et Exécuter le Script d'Installation de l’Environnement Flask** :
   - Ouvre un terminal.
   - Copie le contenu du premier script (`setup_flask_env.sh`) dans un fichier et rends-le exécutable :
     ```bash
     chmod +x setup_flask_env.sh
     ```
   - Exécute le script :
     ```bash
     ./setup_flask_env.sh
     ```

2. **Créer et Exécuter le Script pour l’Application Flask** :
   - Copie le contenu du deuxième script (`create_flask_app.sh`) dans un fichier et rends-le exécutable :
     ```bash
     chmod +x create_flask_app.sh
     ```
   - Exécute le script :
     ```bash
     ./create_flask_app.sh
     ```

### Conclusion

Ces scripts vous permettront de configurer rapidement un environnement de développement Flask et de créer une application de base. Assurez-vous d’exécuter les scripts dans l’ordre pour éviter tout problème de chemin. Si vous avez besoin de fonctionnalités supplémentaires ou d’autres configurations, n’hésitez pas à demander !