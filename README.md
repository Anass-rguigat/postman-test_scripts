# Guide Postman - API Inventory Management System

Ce document contient tous les scripts Postman pour tester chaque endpoint de l'application.

## Configuration de Base

### Variables d'Environnement Postman

Créez un environnement Postman avec les variables suivantes :

```
base_url: http://localhost:5050
token: (sera rempli automatiquement après login)
categoryId: (sera rempli automatiquement après création d'une catégorie)
supplierId: (sera rempli automatiquement après création d'un fournisseur)
productId: (sera rempli automatiquement après création d'un produit)
```

### Headers Communs

Pour les requêtes authentifiées, ajoutez ce header :
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### ⚠️ IMPORTANT : Ordre d'Exécution des Tests

**Vous DEVEZ suivre cet ordre pour que les tests fonctionnent :**

1. **D'abord, connectez-vous** avec un utilisateur ADMIN :
   - Utilisez l'endpoint **Login** avec les credentials par défaut :
     - Email: `rguigat.anass@gmail.com`
     - Password: `12345678`
   - OU créez un nouvel utilisateur avec **Register** (rôle ADMIN)
   - **Vérifiez que le token est sauvegardé** dans la variable d'environnement `{{token}}`

2. **Ensuite, créez les données de base** :
   - Créez une **Category** (nécessaire pour créer des produits)
   - Créez un **Supplier** (nécessaire pour les transactions d'achat)

3. **Puis testez les autres endpoints** :
   - Products (nécessite une categoryId valide)
   - Transactions (nécessite productId et supplierId valides)
   - Users

**Note** : La plupart des endpoints nécessitent un token JWT valide. Assurez-vous que le token est sauvegardé après le login.

---

### 🔧 Résolution des Problèmes Courants

#### Erreur 401 (Unauthorized)
**Symptômes** : `Status code is 200 | AssertionError: expected response to have status code 200 but got 401`

**Causes possibles** :
- Token manquant ou invalide
- Token expiré
- Header Authorization mal configuré

**Solutions** :
1. Vérifiez que vous avez exécuté le **Login** en premier
2. Vérifiez que le token est sauvegardé dans `{{token}}`
3. Vérifiez que le header `Authorization: Bearer {{token}}` est présent
4. Si le token est expiré, reconnectez-vous

#### Erreur 500 (Internal Server Error)
**Symptômes** : `Status code is 200 | AssertionError: expected response to have status code 200 but got 500`

**Causes possibles** :
- **Pour Products** :
  - `categoryId` invalide ou manquant (créez d'abord une catégorie)
  - Fichier image invalide ou trop volumineux
  - Contraintes de base de données (produit utilisé dans des transactions)
  
- **Pour Suppliers** :
  - Contraintes de base de données (supplier utilisé dans des transactions)
  - Données de validation invalides

- **Pour Users** :
  - Permissions insuffisantes (besoin du rôle ADMIN)
  - Utilisateur utilisé dans des transactions

**Solutions** :
1. Vérifiez les logs du backend pour voir le message d'erreur exact
2. Assurez-vous que les données prérequises existent (catégories, suppliers)
3. Vérifiez que vous utilisez un utilisateur avec le rôle ADMIN
4. Ne supprimez pas des entités qui sont utilisées dans des transactions

#### Erreur "Product Not Found" ou "Supplier Not Found"
**Solutions** :
- Vérifiez que l'ID existe dans la base de données
- Utilisez d'abord les endpoints GET pour lister les entités et obtenir les IDs valides

#### Erreur "Category Not Found" lors de la création d'un produit
**Solutions** :
1. Créez d'abord une catégorie avec l'endpoint `POST /api/categories/add`
2. Sauvegardez l'ID de la catégorie créée
3. Utilisez cet ID dans le champ `categoryId` lors de la création du produit

---

## 📋 Liste Complète de Tous les Endpoints

### 🔐 Authentification
- **POST** `/api/auth/register` - Enregistrer un nouvel utilisateur
- **POST** `/api/auth/login` - Se connecter et obtenir un token JWT

### 📦 Catégories
- **POST** `/api/categories/add` - Créer une catégorie (ADMIN seulement)
- **GET** `/api/categories/all` - Obtenir toutes les catégories
- **GET** `/api/categories/{id}` - Obtenir une catégorie par ID
- **PUT** `/api/categories/update/{id}` - Mettre à jour une catégorie (ADMIN seulement)
- **DELETE** `/api/categories/delete/{id}` - Supprimer une catégorie (ADMIN seulement)

### 🛍️ Produits
- **POST** `/api/products/add` - Créer un produit (ADMIN seulement)
- **PUT** `/api/products/update` - Mettre à jour un produit (ADMIN seulement)
- **GET** `/api/products/all` - Obtenir tous les produits
- **GET** `/api/products/{id}` - Obtenir un produit par ID
- **GET** `/api/products/search?input={term}` - Rechercher des produits
- **DELETE** `/api/products/delete/{id}` - Supprimer un produit (ADMIN seulement)

### 🏢 Fournisseurs
- **POST** `/api/suppliers/add` - Ajouter un fournisseur (ADMIN seulement)
- **GET** `/api/suppliers/all` - Obtenir tous les fournisseurs
- **GET** `/api/suppliers/{id}` - Obtenir un fournisseur par ID
- **PUT** `/api/suppliers/update/{id}` - Mettre à jour un fournisseur (ADMIN seulement)
- **DELETE** `/api/suppliers/delete/{id}` - Supprimer un fournisseur (ADMIN seulement)

### 💰 Transactions
- **POST** `/api/transactions/purchase` - Créer une transaction d'achat
- **POST** `/api/transactions/sell` - Créer une transaction de vente
- **POST** `/api/transactions/return` - Retourner un produit au fournisseur
- **GET** `/api/transactions/all?page={page}&size={size}&filter={filter}` - Obtenir toutes les transactions
- **GET** `/api/transactions/{id}` - Obtenir une transaction par ID
- **GET** `/api/transactions/by-month-year?month={month}&year={year}` - Obtenir les transactions par mois/année
- **PUT** `/api/transactions/{transactionId}` - Mettre à jour le statut d'une transaction

### 👥 Utilisateurs
- **GET** `/api/users/all` - Obtenir tous les utilisateurs (ADMIN seulement)
- **GET** `/api/users/{id}` - Obtenir un utilisateur par ID
- **GET** `/api/users/current` - Obtenir l'utilisateur actuellement connecté
- **GET** `/api/users/transactions/{userId}` - Obtenir les transactions d'un utilisateur
- **PUT** `/api/users/update/{id}` - Mettre à jour un utilisateur
- **DELETE** `/api/users/delete/{id}` - Supprimer un utilisateur (ADMIN seulement)

---

## 1. Authentification

### 1.1. Register User

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/auth/register`  
**URL avec variable** : `{{base_url}}/api/auth/register`  
**Headers** : 
```
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "name": "John Doe",
    "email": "john.doe@example.com",
    "password": "password123",
    "phoneNumber": "1234567890",
    "role": "ADMIN"
}
```

**Exemple avec rôle MANAGER** :
```json
{
    "name": "Jane Smith",
    "email": "jane.smith@example.com",
    "password": "password456",
    "phoneNumber": "0987654321",
    "role": "MANAGER"
}
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "User was successfully registered",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
if (pm.response.code === 200) {
    pm.test("Response has status and message", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property('status');
        pm.expect(jsonData).to.have.property('message');
    });

    // Test 4: Vérifier le message de succès
    pm.test("Registration message is correct", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.status).to.eql(200);
        pm.expect(jsonData.message).to.eql("User was successfully registered");
    });
} else {
    // Gérer les erreurs
    pm.test("Check error response", function () {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    });
}

// Test 5: Vérifier le temps de réponse
pm.test("Response time is less than 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

---

### 1.2. Login User

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/auth/login`  
**URL avec variable** : `{{base_url}}/api/auth/login`  
**Headers** : 
```
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "email": "john.doe@example.com",
    "password": "password123"
}
```

**Exemple avec autre utilisateur** :
```json
{
    "email": "jane.smith@example.com",
    "password": "password456"
}
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "User Logged in Successfully",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "role": "MANAGER",
    "expirationTime": "6 months"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la présence du token
pm.test("Response contains token", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('token');
    pm.expect(jsonData.token).to.be.a('string');
    pm.expect(jsonData.token.length).to.be.above(0);
});

// Test 4: Sauvegarder le token dans l'environnement
pm.test("Save token to environment", function () {
    var jsonData = pm.response.json();
    if (jsonData.token) {
        pm.environment.set("token", jsonData.token);
        pm.environment.set("userRole", jsonData.role);
        console.log("Token saved:", jsonData.token);
        console.log("User role:", jsonData.role);
    }
});

// Test 5: Vérifier le message de succès
pm.test("Login message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("User Logged in Successfully");
    pm.expect(jsonData.role).to.be.a('string');
    pm.expect(jsonData.expirationTime).to.eql("6 months");
});

// Test 6: Vérifier le format du token JWT
pm.test("Token is valid JWT format", function () {
    var jsonData = pm.response.json();
    var token = jsonData.token;
    var parts = token.split('.');
    pm.expect(parts.length).to.eql(3);
});
```

---

## 2. Categories

### 2.1. Create Category (ADMIN only)

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/categories/add`  
**URL avec variable** : `{{base_url}}/api/categories/add`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "name": "Electronics"
}
```

**Autres exemples** :
```json
{
    "name": "Clothing"
}
```

```json
{
    "name": "Food & Beverages"
}
```

```json
{
    "name": "Home & Garden"
}
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Category Saved Successfully",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});

// Test 4: Vérifier le message de succès
pm.test("Category creation message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Category Saved Successfully");
});

// Test 5: Sauvegarder l'ID de la catégorie créée
pm.test("Save category ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.category && jsonData.category.id) {
        pm.environment.set("categoryId", jsonData.category.id);
        console.log("Category ID saved:", jsonData.category.id);
    }
});
```

---

### 2.2. Get All Categories

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/categories/all`  
**URL avec variable** : `{{base_url}}/api/categories/all`  
**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "categories": [
        {
            "id": 1,
            "name": "Electronics"
        },
        {
            "id": 2,
            "name": "Clothing"
        }
    ],
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains categories array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData).to.have.property('categories');
    pm.expect(jsonData.categories).to.be.an('array');
});

// Test 4: Vérifier le message de succès
pm.test("Success message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("success");
});

// Test 5: Vérifier la structure des catégories
pm.test("Categories have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.categories && jsonData.categories.length > 0) {
        var category = jsonData.categories[0];
        pm.expect(category).to.have.property('id');
        pm.expect(category).to.have.property('name');
        pm.expect(category.id).to.be.a('number');
        pm.expect(category.name).to.be.a('string');
    }
});
```

---

### 2.3. Get Category By ID

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/categories/1`  
**URL avec variable** : `{{base_url}}/api/categories/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/categories/1`
- `http://localhost:5050/api/categories/2`
- `{{base_url}}/api/categories/{{categoryId}}`

**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "category": {
        "id": 1,
        "name": "Electronics",
        "products": []
    },
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains category object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('category');
    pm.expect(jsonData.category).to.be.an('object');
});

// Test 4: Vérifier les champs de la catégorie
pm.test("Category has required fields", function () {
    var jsonData = pm.response.json();
    var category = jsonData.category;
    pm.expect(category).to.have.property('id');
    pm.expect(category).to.have.property('name');
    pm.expect(category.id).to.be.a('number');
    pm.expect(category.name).to.be.a('string');
});

// Test 5: Vérifier que l'ID correspond
pm.test("Category ID matches request", function () {
    var jsonData = pm.response.json();
    var urlParts = pm.request.url.toString().split('/');
    var requestedId = urlParts[urlParts.length - 1];
    pm.expect(jsonData.category.id.toString()).to.eql(requestedId);
});
```

---

### 2.4. Update Category (ADMIN only)

**Méthode** : `PUT`  
**URL complète** : `http://localhost:5050/api/categories/update/1`  
**URL avec variable** : `{{base_url}}/api/categories/update/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/categories/update/1`
- `{{base_url}}/api/categories/update/{{categoryId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "name": "Updated Electronics"
}
```

**Autre exemple** :
```json
{
    "name": "Electronics & Technology"
}
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Category Was Successfully Updated",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Category Was Successfully Updated");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

### 2.5. Delete Category (ADMIN only)

**Méthode** : `DELETE`  
**URL complète** : `http://localhost:5050/api/categories/delete/1`  
**URL avec variable** : `{{base_url}}/api/categories/delete/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/categories/delete/1`
- `{{base_url}}/api/categories/delete/{{categoryId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Category Was Successfully Deleted",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Delete message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Category Was Successfully Deleted");
});

// Test 4: Vérifier que la catégorie est supprimée
pm.test("Category should be deleted", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
});
```

---

## 3. Products

### 3.1. Create Product (ADMIN only)

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/products/add`  
**URL avec variable** : `{{base_url}}/api/products/add`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: multipart/form-data
```

**Body (form-data)** :
```
Key: imageFile          Type: File          Value: [Sélectionner un fichier image]
Key: name               Type: Text          Value: Laptop
Key: sku                Type: Text          Value: LAP001
Key: price              Type: Text          Value: 999.99
Key: stockQuantity      Type: Text          Value: 10
Key: categoryId         Type: Text          Value: 1
Key: description        Type: Text          Value: High performance laptop
```

**Exemple complet avec données** :
```
imageFile: [Fichier: laptop.jpg]
name: Laptop Dell XPS 15
sku: LAP-DELL-XPS15-001
price: 1299.99
stockQuantity: 25
categoryId: 1
description: High performance laptop with Intel i7 processor, 16GB RAM, 512GB SSD
```

**Autre exemple** :
```
imageFile: [Fichier: smartphone.jpg]
name: iPhone 15 Pro
sku: PHN-IPHONE15PRO-001
price: 999.99
stockQuantity: 50
categoryId: 1
description: Latest iPhone with A17 Pro chip and titanium design
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Product successfully saved",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("ERROR: No token found. Please login first!");
    throw new Error("Token required. Please login first.");
}

// Vérifier si categoryId existe
if (!pm.environment.get("categoryId")) {
    console.log("WARNING: No categoryId found. Make sure to create a category first!");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    if (pm.response.code !== 200) {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    }
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès (seulement si succès)
if (pm.response.code === 200) {
    pm.test("Product creation message is correct", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.status).to.eql(200);
        pm.expect(jsonData.message).to.eql("Product successfully saved");
    });

    // Test 4: Sauvegarder l'ID du produit créé
    pm.test("Save product ID if returned", function () {
        var jsonData = pm.response.json();
        if (jsonData.product && jsonData.product.id) {
            pm.environment.set("productId", jsonData.product.id);
            console.log("Product ID saved:", jsonData.product.id);
        }
    });
} else {
    // Afficher les détails de l'erreur
    pm.test("Log error details", function () {
        var jsonData = pm.response.json();
        console.log("❌ Error creating product:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Possible causes:");
        console.log("- Invalid categoryId (create a category first)");
        console.log("- Missing required fields");
        console.log("- Invalid image file");
        console.log("- Database connection issue");
    });
}
```

---

### 3.2. Update Product (ADMIN only)

**Méthode** : `PUT`  
**URL complète** : `http://localhost:5050/api/products/update`  
**URL avec variable** : `{{base_url}}/api/products/update`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: multipart/form-data
```

**Body (form-data)** :
```
Key: productId          Type: Text          Value: 1
Key: imageFile         Type: File          Value: [Fichier image optionnel - peut être vide]
Key: name               Type: Text          Value: Updated Laptop
Key: sku                Type: Text          Value: LAP001-UPDATED
Key: price              Type: Text          Value: 1299.99
Key: stockQuantity      Type: Text          Value: 15
Key: categoryId         Type: Text          Value: 1
Key: description        Type: Text          Value: Updated description with new features
```

**Exemple complet** :
```
productId: 1
imageFile: [Fichier: laptop-updated.jpg] (optionnel)
name: Laptop Dell XPS 15 Updated
sku: LAP-DELL-XPS15-001
price: 1199.99
stockQuantity: 20
categoryId: 1
description: Updated laptop with improved specifications
```

**Note** : Tous les champs sauf `productId` sont optionnels. Vous pouvez mettre à jour seulement certains champs.

**Response attendue** :
```json
{
    "status": 200,
    "message": "Product Updated successfully",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Vérifier si productId existe
if (!pm.environment.get("productId")) {
    console.log("Warning: No productId found. Using default value 1");
    pm.environment.set("productId", "1");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Product update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product Updated successfully");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});

// Test 5: Vérifier que le produit a été mis à jour
pm.test("Product was updated successfully", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    console.log("Product updated successfully");
});
```

---

### 3.3. Get All Products

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/products/all`  
**URL avec variable** : `{{base_url}}/api/products/all`  
**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "products": [
        {
            "id": 1,
            "name": "Laptop",
            "sku": "LAP001",
            "price": 999.99,
            "stockQuantity": 10,
            "description": "High performance laptop",
            "imageUrl": "products/uuid_filename.jpg",
            "categoryId": 1
        }
    ],
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains products array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('products');
    pm.expect(jsonData.products).to.be.an('array');
});

// Test 4: Vérifier la structure des produits
pm.test("Products have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.products && jsonData.products.length > 0) {
        var product = jsonData.products[0];
        pm.expect(product).to.have.property('id');
        pm.expect(product).to.have.property('name');
        pm.expect(product).to.have.property('sku');
        pm.expect(product).to.have.property('price');
        pm.expect(product).to.have.property('stockQuantity');
    }
});
```

---

### 3.4. Get Product By ID

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/products/1`  
**URL avec variable** : `{{base_url}}/api/products/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/products/1`
- `http://localhost:5050/api/products/2`
- `{{base_url}}/api/products/{{productId}}`

**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "product": {
        "id": 1,
        "name": "Laptop",
        "sku": "LAP001",
        "price": 999.99,
        "stockQuantity": 10,
        "description": "High performance laptop",
        "imageUrl": "products/uuid_filename.jpg",
        "categoryId": 1
    },
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains product object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('product');
    pm.expect(jsonData.product).to.be.an('object');
});

// Test 4: Vérifier les champs du produit
pm.test("Product has required fields", function () {
    var jsonData = pm.response.json();
    var product = jsonData.product;
    pm.expect(product).to.have.property('id');
    pm.expect(product).to.have.property('name');
    pm.expect(product).to.have.property('sku');
    pm.expect(product).to.have.property('price');
    pm.expect(product).to.have.property('stockQuantity');
});
```

---

### 3.5. Delete Product (ADMIN only)

**Méthode** : `DELETE`  
**URL complète** : `http://localhost:5050/api/products/delete/1`  
**URL avec variable** : `{{base_url}}/api/products/delete/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/products/delete/1`
- `{{base_url}}/api/products/delete/{{productId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Product Deleted successfully",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("ERROR: No token found. Please login first!");
    throw new Error("Token required. Please login first.");
}

// Vérifier si productId existe
if (!pm.environment.get("productId")) {
    console.log("WARNING: No productId found. Make sure to create a product first!");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    if (pm.response.code !== 200) {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    }
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès (seulement si succès)
if (pm.response.code === 200) {
    pm.test("Product delete message is correct", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.status).to.eql(200);
        pm.expect(jsonData.message).to.eql("Product Deleted successfully");
    });

    // Test 4: Vérifier la structure de la réponse
    pm.test("Response has status and message", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property('status');
        pm.expect(jsonData).to.have.property('message');
    });

    // Test 5: Vérifier que le produit est supprimé
    pm.test("Product should be deleted", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.status).to.eql(200);
        console.log("✅ Product deleted successfully");
    });
} else {
    // Afficher les détails de l'erreur
    pm.test("Log error details", function () {
        var jsonData = pm.response.json();
        console.log("❌ Error deleting product:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Possible causes:");
        console.log("- Product not found (ID doesn't exist)");
        console.log("- Product is used in transactions (foreign key constraint)");
        console.log("- Missing authentication token");
        console.log("- Insufficient permissions (need ADMIN role)");
    });
}
```

---

### 3.6. Search Product

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/products/search?input=Laptop`  
**URL avec variable** : `{{base_url}}/api/products/search?input=Laptop`  
**Exemples d'URLs** :
- `http://localhost:5050/api/products/search?input=Laptop`
- `http://localhost:5050/api/products/search?input=iPhone`
- `http://localhost:5050/api/products/search?input=LAP001`
- `{{base_url}}/api/products/search?input={{searchTerm}}`

**Query Parameters** :
- `input` : Terme de recherche (requis)

**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "products": [
        {
            "id": 1,
            "name": "Laptop",
            "sku": "LAP001",
            "price": 999.99,
            "stockQuantity": 10,
            "description": "High performance laptop",
            "categoryId": 1
        }
    ],
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains products array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('products');
    pm.expect(jsonData.products).to.be.an('array');
});

// Test 4: Vérifier que les résultats correspondent à la recherche
pm.test("Search results match query", function () {
    var jsonData = pm.response.json();
    var searchInput = pm.request.url.query.get("input");
    if (jsonData.products && jsonData.products.length > 0) {
        var product = jsonData.products[0];
        var nameMatch = product.name.toLowerCase().includes(searchInput.toLowerCase());
        var descMatch = product.description && product.description.toLowerCase().includes(searchInput.toLowerCase());
        pm.expect(nameMatch || descMatch).to.be.true;
    }
});
```

---

## 4. Suppliers

### 4.1. Add Supplier (ADMIN only)

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/suppliers/add`  
**URL avec variable** : `{{base_url}}/api/suppliers/add`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "name": "Tech Supplier Inc",
    "contactInfo": "contact@techsupplier.com",
    "address": "123 Tech Street, City, Country"
}
```

**Autres exemples** :
```json
{
    "name": "Global Electronics Ltd",
    "contactInfo": "sales@globalelectronics.com",
    "address": "456 Business Avenue, New York, USA"
}
```

```json
{
    "name": "Fast Delivery Co",
    "contactInfo": "+1-555-0123",
    "address": "789 Warehouse Road, Los Angeles, USA"
}
```

**Note** : `address` est optionnel, mais `name` et `contactInfo` sont requis.

**Response attendue** :
```json
{
    "status": 200,
    "message": "Supplier Saved Successfully",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Supplier creation message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Supplier Saved Successfully");
});

// Test 4: Sauvegarder l'ID du fournisseur créé
pm.test("Save supplier ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.supplier && jsonData.supplier.id) {
        pm.environment.set("supplierId", jsonData.supplier.id);
        console.log("Supplier ID saved:", jsonData.supplier.id);
    }
});
```

---

### 4.2. Get All Suppliers

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/suppliers/all`  
**URL avec variable** : `{{base_url}}/api/suppliers/all`  
**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "suppliers": [
        {
            "id": 1,
            "name": "Tech Supplier Inc",
            "contactInfo": "contact@techsupplier.com",
            "address": "123 Tech Street, City, Country"
        }
    ],
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains suppliers array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('suppliers');
    pm.expect(jsonData.suppliers).to.be.an('array');
});

// Test 4: Vérifier la structure des fournisseurs
pm.test("Suppliers have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.suppliers && jsonData.suppliers.length > 0) {
        var supplier = jsonData.suppliers[0];
        pm.expect(supplier).to.have.property('id');
        pm.expect(supplier).to.have.property('name');
        pm.expect(supplier).to.have.property('contactInfo');
    }
});
```

---

### 4.3. Get Supplier By ID

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/suppliers/1`  
**URL avec variable** : `{{base_url}}/api/suppliers/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/suppliers/1`
- `{{base_url}}/api/suppliers/{{supplierId}}`

**Headers** : 
```
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "supplier": {
        "id": 1,
        "name": "Tech Supplier Inc",
        "contactInfo": "contact@techsupplier.com",
        "address": "123 Tech Street, City, Country"
    },
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains supplier object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('supplier');
    pm.expect(jsonData.supplier).to.be.an('object');
});

// Test 4: Vérifier les champs du fournisseur
pm.test("Supplier has required fields", function () {
    var jsonData = pm.response.json();
    var supplier = jsonData.supplier;
    pm.expect(supplier).to.have.property('id');
    pm.expect(supplier).to.have.property('name');
    pm.expect(supplier).to.have.property('contactInfo');
});
```

---

### 4.4. Update Supplier (ADMIN only)

**Méthode** : `PUT`  
**URL complète** : `http://localhost:5050/api/suppliers/update/1`  
**URL avec variable** : `{{base_url}}/api/suppliers/update/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/suppliers/update/1`
- `{{base_url}}/api/suppliers/update/{{supplierId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "name": "Updated Tech Supplier",
    "contactInfo": "newcontact@techsupplier.com",
    "address": "456 New Street, City, Country"
}
```

**Exemple avec mise à jour partielle** :
```json
{
    "name": "Tech Supplier Inc - Updated",
    "contactInfo": "info@techsupplier.com"
}
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Supplier Was Successfully Updated",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Supplier update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Supplier Was Successfully Updated");
});
```

---

### 4.5. Delete Supplier (ADMIN only)

**Méthode** : `DELETE`  
**URL complète** : `http://localhost:5050/api/suppliers/delete/1`  
**URL avec variable** : `{{base_url}}/api/suppliers/delete/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/suppliers/delete/1`
- `{{base_url}}/api/suppliers/delete/{{supplierId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Supplier Was Successfully Deleted",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("ERROR: No token found. Please login first!");
    throw new Error("Token required. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    if (pm.response.code !== 200) {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    }
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès (seulement si succès)
if (pm.response.code === 200) {
    pm.test("Supplier delete message is correct", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.status).to.eql(200);
        pm.expect(jsonData.message).to.eql("Supplier Was Successfully Deleted");
    });
} else {
    // Afficher les détails de l'erreur
    pm.test("Log error details", function () {
        var jsonData = pm.response.json();
        console.log("❌ Error deleting supplier:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Possible causes:");
        console.log("- Supplier not found (ID doesn't exist)");
        console.log("- Supplier is used in transactions (foreign key constraint)");
        console.log("- Missing authentication token");
        console.log("- Insufficient permissions (need ADMIN role)");
    });
}
```

---

## 5. Transactions

### 5.1. Purchase Transaction

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/transactions/purchase`  
**URL avec variable** : `{{base_url}}/api/transactions/purchase`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "productId": 1,
    "supplierId": 1,
    "quantity": 5,
    "description": "Purchase of laptops from supplier",
    "note": "Bulk purchase order"
}
```

**Autres exemples** :
```json
{
    "productId": 1,
    "supplierId": 1,
    "quantity": 10,
    "description": "Monthly stock replenishment",
    "note": "Regular order"
}
```

```json
{
    "productId": 2,
    "supplierId": 2,
    "quantity": 20,
    "description": "Emergency purchase for high demand",
    "note": "Urgent order"
}
```

**Note** : `description` et `note` sont optionnels, mais `productId`, `supplierId` et `quantity` sont requis.

**Response attendue** :
```json
{
    "status": 200,
    "message": "Purchase Made successfully",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Purchase message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Purchase Made successfully");
});

// Test 4: Sauvegarder l'ID de la transaction créée
pm.test("Save transaction ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.transaction && jsonData.transaction.id) {
        pm.environment.set("transactionId", jsonData.transaction.id);
        console.log("Transaction ID saved:", jsonData.transaction.id);
    }
});
```

---

### 5.2. Sell Transaction

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/transactions/sell`  
**URL avec variable** : `{{base_url}}/api/transactions/sell`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "productId": 1,
    "quantity": 2,
    "description": "Sale to customer",
    "note": "Customer purchase"
}
```

**Autres exemples** :
```json
{
    "productId": 1,
    "quantity": 1,
    "description": "Retail sale",
    "note": "Walk-in customer"
}
```

```json
{
    "productId": 2,
    "quantity": 5,
    "description": "Bulk sale to corporate client",
    "note": "Corporate order #12345"
}
```

**Note** : `supplierId` n'est pas requis pour les ventes. `productId` et `quantity` sont requis.

**Response attendue** :
```json
{
    "status": 200,
    "message": "Product Sale successfully made",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Sell message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product Sale successfully made");
});
```

---

### 5.3. Return to Supplier

**Méthode** : `POST`  
**URL complète** : `http://localhost:5050/api/transactions/return`  
**URL avec variable** : `{{base_url}}/api/transactions/return`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "productId": 1,
    "supplierId": 1,
    "quantity": 1,
    "description": "Return defective product",
    "note": "Defective item return"
}
```

**Autres exemples** :
```json
{
    "productId": 1,
    "supplierId": 1,
    "quantity": 3,
    "description": "Return damaged goods",
    "note": "Items damaged during shipping"
}
```

```json
{
    "productId": 2,
    "supplierId": 2,
    "quantity": 2,
    "description": "Return wrong items",
    "note": "Received wrong product model"
}
```

**Note** : `productId`, `supplierId` et `quantity` sont requis pour les retours.

**Response attendue** :
```json
{
    "status": 200,
    "message": "Product Returned in progress",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Return message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product Returned in progress");
});
```

---

### 5.4. Get All Transactions

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/transactions/all?page=0&size=10&filter=purchase`  
**URL avec variable** : `{{base_url}}/api/transactions/all?page=0&size=10&filter=purchase`  
**Exemples d'URLs** :
- `http://localhost:5050/api/transactions/all` (toutes les transactions)
- `http://localhost:5050/api/transactions/all?page=0&size=10` (première page, 10 éléments)
- `http://localhost:5050/api/transactions/all?page=1&size=20` (deuxième page, 20 éléments)
- `http://localhost:5050/api/transactions/all?filter=purchase` (filtrer par "purchase")
- `http://localhost:5050/api/transactions/all?filter=sell` (filtrer par "sell")
- `{{base_url}}/api/transactions/all?page={{page}}&size={{size}}&filter={{filter}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Query Parameters** :
- `page`: Numéro de page (défaut: 0) - Exemple: 0, 1, 2
- `size`: Taille de la page (défaut: 1000) - Exemple: 10, 20, 50
- `filter`: Filtre de recherche (optionnel) - Exemple: "purchase", "sell", "return"

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "transactions": [
        {
            "id": 1,
            "totalProducts": 5,
            "totalPrice": 4999.95,
            "transactionType": "PURCHASE",
            "status": "COMPLETED",
            "description": "Purchase of laptops",
            "note": "Bulk purchase",
            "createdAt": "2024-01-15T10:30:00"
        }
    ],
    "totalPages": 1,
    "totalElements": 1,
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains transactions array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('transactions');
    pm.expect(jsonData.transactions).to.be.an('array');
});

// Test 4: Vérifier la pagination
pm.test("Response contains pagination info", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('totalPages');
    pm.expect(jsonData).to.have.property('totalElements');
    pm.expect(jsonData.totalPages).to.be.a('number');
    pm.expect(jsonData.totalElements).to.be.a('number');
});

// Test 5: Vérifier la structure des transactions
pm.test("Transactions have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.transactions && jsonData.transactions.length > 0) {
        var transaction = jsonData.transactions[0];
        pm.expect(transaction).to.have.property('id');
        pm.expect(transaction).to.have.property('transactionType');
        pm.expect(transaction).to.have.property('status');
        pm.expect(transaction).to.have.property('totalProducts');
        pm.expect(transaction).to.have.property('totalPrice');
    }
});
```

---

### 5.5. Get Transaction By ID

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/transactions/1`  
**URL avec variable** : `{{base_url}}/api/transactions/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/transactions/1`
- `http://localhost:5050/api/transactions/2`
- `{{base_url}}/api/transactions/{{transactionId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "transaction": {
        "id": 1,
        "totalProducts": 5,
        "totalPrice": 4999.95,
        "transactionType": "PURCHASE",
        "status": "COMPLETED",
        "description": "Purchase of laptops",
        "note": "Bulk purchase",
        "createdAt": "2024-01-15T10:30:00",
        "user": {
            "id": 1,
            "name": "John Doe",
            "email": "john.doe@example.com"
        },
        "product": {
            "id": 1,
            "name": "Laptop",
            "sku": "LAP001"
        },
        "supplier": {
            "id": 1,
            "name": "Tech Supplier Inc"
        }
    },
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains transaction object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('transaction');
    pm.expect(jsonData.transaction).to.be.an('object');
});

// Test 4: Vérifier les champs de la transaction
pm.test("Transaction has required fields", function () {
    var jsonData = pm.response.json();
    var transaction = jsonData.transaction;
    pm.expect(transaction).to.have.property('id');
    pm.expect(transaction).to.have.property('transactionType');
    pm.expect(transaction).to.have.property('status');
    pm.expect(transaction).to.have.property('totalProducts');
    pm.expect(transaction).to.have.property('totalPrice');
});
```

---

### 5.6. Get Transactions By Month and Year

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/transactions/by-month-year?month=1&year=2024`  
**URL avec variable** : `{{base_url}}/api/transactions/by-month-year?month=1&year=2024`  
**Exemples d'URLs** :
- `http://localhost:5050/api/transactions/by-month-year?month=1&year=2024` (Janvier 2024)
- `http://localhost:5050/api/transactions/by-month-year?month=12&year=2024` (Décembre 2024)
- `http://localhost:5050/api/transactions/by-month-year?month=3&year=2023` (Mars 2023)
- `{{base_url}}/api/transactions/by-month-year?month={{month}}&year={{year}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Query Parameters** :
- `month`: Mois (1-12) - Requis
- `year`: Année (ex: 2024) - Requis

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "transactions": [
        {
            "id": 1,
            "totalProducts": 5,
            "totalPrice": 4999.95,
            "transactionType": "PURCHASE",
            "status": "COMPLETED",
            "createdAt": "2024-01-15T10:30:00"
        }
    ],
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains transactions array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('transactions');
    pm.expect(jsonData.transactions).to.be.an('array');
});

// Test 4: Vérifier que les transactions correspondent au mois/année demandé
pm.test("Transactions match requested month and year", function () {
    var jsonData = pm.response.json();
    var month = parseInt(pm.request.url.query.get("month"));
    var year = parseInt(pm.request.url.query.get("year"));
    
    if (jsonData.transactions && jsonData.transactions.length > 0) {
        var transaction = jsonData.transactions[0];
        if (transaction.createdAt) {
            var transactionDate = new Date(transaction.createdAt);
            pm.expect(transactionDate.getMonth() + 1).to.eql(month);
            pm.expect(transactionDate.getFullYear()).to.eql(year);
        }
    }
});
```

---

### 5.7. Update Transaction Status

**Méthode** : `PUT`  
**URL complète** : `http://localhost:5050/api/transactions/1`  
**URL avec variable** : `{{base_url}}/api/transactions/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/transactions/1`
- `{{base_url}}/api/transactions/{{transactionId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
"COMPLETED"
```

**Valeurs possibles pour le statut** :
- `"PENDING"` - En attente
- `"PROCESSING"` - En cours de traitement
- `"COMPLETED"` - Terminé
- `"CANCELLED"` - Annulé

**Exemples** :
```json
"PENDING"
```

```json
"PROCESSING"
```

```json
"CANCELLED"
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "Transaction Status Successfully Updated",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Transaction status update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Transaction Status Successfully Updated");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

## 6. Users

### 6.1. Get All Users (ADMIN only)

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/users/all`  
**URL avec variable** : `{{base_url}}/api/users/all`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "users": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john.doe@example.com",
            "phoneNumber": "1234567890",
            "role": "MANAGER",
            "createdAt": "2024-01-15T10:30:00"
        }
    ],
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains users array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('users');
    pm.expect(jsonData.users).to.be.an('array');
});

// Test 4: Vérifier la structure des utilisateurs
pm.test("Users have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.users && jsonData.users.length > 0) {
        var user = jsonData.users[0];
        pm.expect(user).to.have.property('id');
        pm.expect(user).to.have.property('name');
        pm.expect(user).to.have.property('email');
        pm.expect(user).to.have.property('role');
    }
});
```

---

### 6.2. Get User By ID

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/users/1`  
**URL avec variable** : `{{base_url}}/api/users/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/users/1`
- `{{base_url}}/api/users/{{userId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "user": {
        "id": 1,
        "name": "John Doe",
        "email": "john.doe@example.com",
        "phoneNumber": "1234567890",
        "role": "MANAGER",
        "createdAt": "2024-01-15T10:30:00"
    },
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains user object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('user');
    pm.expect(jsonData.user).to.be.an('object');
});

// Test 4: Vérifier les champs de l'utilisateur
pm.test("User has required fields", function () {
    var jsonData = pm.response.json();
    var user = jsonData.user;
    pm.expect(user).to.have.property('id');
    pm.expect(user).to.have.property('name');
    pm.expect(user).to.have.property('email');
    pm.expect(user).to.have.property('role');
});

// Test 5: Vérifier que l'ID correspond
pm.test("User ID matches request", function () {
    var jsonData = pm.response.json();
    var urlParts = pm.request.url.toString().split('/');
    var requestedId = urlParts[urlParts.length - 1];
    pm.expect(jsonData.user.id.toString()).to.eql(requestedId);
});
```

---

### 6.3. Update User

**Méthode** : `PUT`  
**URL complète** : `http://localhost:5050/api/users/update/1`  
**URL avec variable** : `{{base_url}}/api/users/update/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/users/update/1`
- `{{base_url}}/api/users/update/{{userId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body (JSON)** :
```json
{
    "name": "John Updated",
    "email": "john.updated@example.com",
    "phoneNumber": "9876543210",
    "role": "ADMIN",
    "password": "newPassword123"
}
```

**Exemple avec mise à jour partielle** :
```json
{
    "name": "John Doe Updated",
    "phoneNumber": "1234567890"
}
```

**Note** : Tous les champs sont optionnels. Vous pouvez mettre à jour seulement certains champs.

**Response attendue** :
```json
{
    "status": 200,
    "message": "User successfully updated",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("User update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("User successfully updated");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

### 6.4. Delete User (ADMIN only)

**Méthode** : `DELETE`  
**URL complète** : `http://localhost:5050/api/users/delete/1`  
**URL avec variable** : `{{base_url}}/api/users/delete/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/users/delete/1`
- `{{base_url}}/api/users/delete/{{userId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "User successfully Deleted",
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("ERROR: No token found. Please login first!");
    throw new Error("Token required. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    if (pm.response.code !== 200) {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    }
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès (seulement si succès)
if (pm.response.code === 200) {
    pm.test("User delete message is correct", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.status).to.eql(200);
        pm.expect(jsonData.message).to.eql("User successfully Deleted");
    });

    // Test 4: Vérifier la structure de la réponse
    pm.test("Response has status and message", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property('status');
        pm.expect(jsonData).to.have.property('message');
    });
} else if (pm.response.code === 401) {
    // Erreur d'authentification
    pm.test("Authentication error - Token required", function () {
        var jsonData = pm.response.json();
        console.log("❌ Authentication Error:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Solution: Make sure you are logged in and the token is valid");
    });
} else {
    // Autres erreurs
    pm.test("Log error details", function () {
        var jsonData = pm.response.json();
        console.log("❌ Error deleting user:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Possible causes:");
        console.log("- User not found (ID doesn't exist)");
        console.log("- Missing authentication token");
        console.log("- Insufficient permissions (need ADMIN role)");
    });
}
```

---

### 6.5. Get User Transactions

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/users/transactions/1`  
**URL avec variable** : `{{base_url}}/api/users/transactions/1`  
**Exemples d'URLs** :
- `http://localhost:5050/api/users/transactions/1`
- `{{base_url}}/api/users/transactions/{{userId}}`

**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Response attendue** :
```json
{
    "status": 200,
    "message": "success",
    "user": {
        "id": 1,
        "name": "John Doe",
        "email": "john.doe@example.com",
        "transactions": [
            {
                "id": 1,
                "totalProducts": 5,
                "totalPrice": 4999.95,
                "transactionType": "PURCHASE",
                "status": "COMPLETED",
                "createdAt": "2024-01-15T10:30:00"
            }
        ]
    },
    "timestamp": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("ERROR: No token found. Please login first!");
    throw new Error("Token required. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    if (pm.response.code !== 200) {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    }
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse (seulement si succès)
if (pm.response.code === 200) {
    pm.test("Response contains user and transactions", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property('status');
        pm.expect(jsonData).to.have.property('user');
        pm.expect(jsonData.user).to.have.property('transactions');
        pm.expect(jsonData.user.transactions).to.be.an('array');
    });

    // Test 4: Vérifier les champs de l'utilisateur
    pm.test("User has required fields", function () {
        var jsonData = pm.response.json();
        var user = jsonData.user;
        pm.expect(user).to.have.property('id');
        pm.expect(user).to.have.property('name');
        pm.expect(user).to.have.property('email');
    });

    // Test 5: Vérifier la structure des transactions
    pm.test("Transactions have required fields", function () {
        var jsonData = pm.response.json();
        if (jsonData.user.transactions && jsonData.user.transactions.length > 0) {
            var transaction = jsonData.user.transactions[0];
            pm.expect(transaction).to.have.property('id');
            pm.expect(transaction).to.have.property('transactionType');
            pm.expect(transaction).to.have.property('status');
        }
    });
} else if (pm.response.code === 401) {
    // Erreur d'authentification
    pm.test("Authentication error - Token required", function () {
        var jsonData = pm.response.json();
        console.log("❌ Authentication Error:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Solution: Make sure you are logged in and the token is valid");
    });
} else {
    // Autres erreurs
    pm.test("Log error details", function () {
        var jsonData = pm.response.json();
        console.log("❌ Error getting user transactions:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
    });
}
```

---

### 6.6. Get Current User

**Méthode** : `GET`  
**URL complète** : `http://localhost:5050/api/users/current`  
**URL avec variable** : `{{base_url}}/api/users/current`  
**Headers** : 
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Note** : Retourne les informations de l'utilisateur actuellement connecté (basé sur le token JWT).

**Response attendue** :
```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com",
    "phoneNumber": "1234567890",
    "role": "MANAGER",
    "createdAt": "2024-01-15T10:30:00"
}
```

**Scripts Postman (Tests Tab)** :
```javascript
// Pre-request Script: Vérifier si le token existe
if (!pm.environment.get("token")) {
    console.log("ERROR: No token found. Please login first!");
    throw new Error("Token required. Please login first.");
}

// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    if (pm.response.code !== 200) {
        var jsonData = pm.response.json();
        console.log("Error status:", jsonData.status);
        console.log("Error message:", jsonData.message);
    }
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse (seulement si succès)
if (pm.response.code === 200) {
    pm.test("Response contains user object", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property('id');
        pm.expect(jsonData).to.have.property('email');
        pm.expect(jsonData).to.have.property('name');
    });

    // Test 4: Vérifier que l'email correspond au token
    pm.test("User email matches token", function () {
        var jsonData = pm.response.json();
        // L'email devrait correspondre à celui utilisé pour le login
        pm.expect(jsonData.email).to.be.a('string');
        pm.expect(jsonData.email).to.include('@');
    });

    // Test 5: Vérifier les champs requis
    pm.test("User has required fields", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property('role');
        pm.expect(jsonData).to.have.property('phoneNumber');
    });
} else if (pm.response.code === 401) {
    // Erreur d'authentification
    pm.test("Authentication error - Token required", function () {
        var jsonData = pm.response.json();
        console.log("❌ Authentication Error:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
        console.log("Solution: Make sure you are logged in and the token is valid");
        console.log("Steps to fix:");
        console.log("1. Go to Login endpoint");
        console.log("2. Use email: rguigat.anass@gmail.com");
        console.log("3. Use password: 12345678");
        console.log("4. The token will be saved automatically");
    });
} else {
    // Autres erreurs
    pm.test("Log error details", function () {
        var jsonData = pm.response.json();
        console.log("❌ Error getting current user:");
        console.log("Status:", jsonData.status);
        console.log("Message:", jsonData.message);
    });
}
```

---

## Collection Postman

### Import de la Collection

1. Créez une nouvelle collection dans Postman
2. Importez tous les endpoints ci-dessus
3. Configurez les variables d'environnement
4. Exécutez d'abord le login pour obtenir le token
5. Les autres requêtes utiliseront automatiquement le token

### Ordre d'Exécution Recommandé

1. **Register User** → Créer un utilisateur
2. **Login User** → Obtenir le token (sauvegardé automatiquement)
3. **Create Category** → Créer une catégorie
4. **Add Supplier** → Créer un fournisseur
5. **Create Product** → Créer un produit
6. **Purchase Transaction** → Faire un achat
7. **Sell Transaction** → Faire une vente
8. **Get All Transactions** → Voir toutes les transactions

---

## Codes d'Erreur

### 400 Bad Request
```json
{
    "status": 400,
    "message": "Error message",
    "timestamp": "2024-01-15T10:30:00"
}
```

### 401 Unauthorized
```json
{
    "status": 401,
    "message": "Unauthorized",
    "timestamp": "2024-01-15T10:30:00"
}
```

### 403 Forbidden
```json
{
    "status": 403,
    "message": "Access Denied",
    "timestamp": "2024-01-15T10:30:00"
}
```

### 404 Not Found
```json
{
    "status": 404,
    "message": "Resource Not Found",
    "timestamp": "2024-01-15T10:30:00"
}
```

### 500 Internal Server Error
```json
{
    "status": 500,
    "message": "Internal Server Error",
    "timestamp": "2024-01-15T10:30:00"
}
```

---

## Notes Importantes

1. **Authentification** : La plupart des endpoints nécessitent un token JWT valide
2. **Rôles** : Certains endpoints nécessitent le rôle ADMIN
3. **Images** : Les produits peuvent inclure des images (multipart/form-data)
4. **Pagination** : Les transactions utilisent la pagination (page, size)
5. **Filtrage** : Les transactions peuvent être filtrées par texte de recherche
6. **Validation** : Tous les champs requis doivent être fournis

---

---

## Scripts Postman - Tests Automatisés

### Scripts pour chaque Endpoint

#### 1.1. Register User - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData).to.have.property('timestamp');
});

// Test 4: Vérifier le message de succès
pm.test("Registration message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("User was successfully registered");
});

// Test 5: Vérifier le temps de réponse
pm.test("Response time is less than 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

---

#### 1.2. Login User - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la présence du token
pm.test("Response contains token", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('token');
    pm.expect(jsonData.token).to.be.a('string');
    pm.expect(jsonData.token.length).to.be.above(0);
});

// Test 4: Sauvegarder le token dans l'environnement
pm.test("Save token to environment", function () {
    var jsonData = pm.response.json();
    if (jsonData.token) {
        pm.environment.set("token", jsonData.token);
        pm.environment.set("userRole", jsonData.role);
        console.log("Token saved:", jsonData.token);
        console.log("User role:", jsonData.role);
    }
});

// Test 5: Vérifier le message de succès
pm.test("Login message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("User Logged in Successfully");
    pm.expect(jsonData.role).to.be.a('string');
    pm.expect(jsonData.expirationTime).to.eql("6 months");
});

// Test 6: Vérifier le format du token JWT
pm.test("Token is valid JWT format", function () {
    var jsonData = pm.response.json();
    var token = jsonData.token;
    var parts = token.split('.');
    pm.expect(parts.length).to.eql(3);
});
```

---

#### 2.1. Create Category - Scripts de Test

**Pre-request Script** :
```javascript
// Vérifier si le token existe
if (!pm.environment.get("token")) {
    pm.test.skip("Token not found - skipping test");
    console.log("Warning: No token found. Please login first.");
}
```

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});

// Test 4: Vérifier le message de succès
pm.test("Category creation message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Category Saved Successfully");
});

// Test 5: Sauvegarder l'ID de la catégorie créée
pm.test("Save category ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.category && jsonData.category.id) {
        pm.environment.set("categoryId", jsonData.category.id);
        console.log("Category ID saved:", jsonData.category.id);
    }
});
```

---

#### 2.2. Get All Categories - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains categories array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData).to.have.property('categories');
    pm.expect(jsonData.categories).to.be.an('array');
});

// Test 4: Vérifier le message de succès
pm.test("Success message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("success");
});

// Test 5: Vérifier la structure des catégories
pm.test("Categories have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.categories && jsonData.categories.length > 0) {
        var category = jsonData.categories[0];
        pm.expect(category).to.have.property('id');
        pm.expect(category).to.have.property('name');
        pm.expect(category.id).to.be.a('number');
        pm.expect(category.name).to.be.a('string');
    }
});
```

---

#### 2.3. Get Category By ID - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains category object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('category');
    pm.expect(jsonData.category).to.be.an('object');
});

// Test 4: Vérifier les champs de la catégorie
pm.test("Category has required fields", function () {
    var jsonData = pm.response.json();
    var category = jsonData.category;
    pm.expect(category).to.have.property('id');
    pm.expect(category).to.have.property('name');
    pm.expect(category.id).to.be.a('number');
    pm.expect(category.name).to.be.a('string');
});

// Test 5: Vérifier que l'ID correspond
pm.test("Category ID matches request", function () {
    var jsonData = pm.response.json();
    var urlParts = pm.request.url.toString().split('/');
    var requestedId = urlParts[urlParts.length - 1];
    pm.expect(jsonData.category.id.toString()).to.eql(requestedId);
});
```

---

#### 2.4. Update Category - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Category Was Successfully Updated");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

#### 2.5. Delete Category - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Delete message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Category Was Successfully Deleted");
});

// Test 4: Vérifier que la catégorie est supprimée (tentative de récupération)
pm.test("Category should be deleted", function () {
    // Ce test vérifie que la suppression a réussi
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
});
```

---

#### 3.1. Create Product - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Product creation message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product successfully saved");
});

// Test 4: Sauvegarder l'ID du produit créé
pm.test("Save product ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.product && jsonData.product.id) {
        pm.environment.set("productId", jsonData.product.id);
        console.log("Product ID saved:", jsonData.product.id);
    }
});
```

---

#### 3.2. Get All Products - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains products array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('products');
    pm.expect(jsonData.products).to.be.an('array');
});

// Test 4: Vérifier la structure des produits
pm.test("Products have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.products && jsonData.products.length > 0) {
        var product = jsonData.products[0];
        pm.expect(product).to.have.property('id');
        pm.expect(product).to.have.property('name');
        pm.expect(product).to.have.property('sku');
        pm.expect(product).to.have.property('price');
        pm.expect(product).to.have.property('stockQuantity');
    }
});
```

---

#### 3.3. Get Product By ID - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains product object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('product');
    pm.expect(jsonData.product).to.be.an('object');
});

// Test 4: Vérifier les champs du produit
pm.test("Product has required fields", function () {
    var jsonData = pm.response.json();
    var product = jsonData.product;
    pm.expect(product).to.have.property('id');
    pm.expect(product).to.have.property('name');
    pm.expect(product).to.have.property('sku');
    pm.expect(product).to.have.property('price');
    pm.expect(product).to.have.property('stockQuantity');
});
```

---

#### 3.4. Search Product - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains products array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('products');
    pm.expect(jsonData.products).to.be.an('array');
});

// Test 4: Vérifier que les résultats correspondent à la recherche
pm.test("Search results match query", function () {
    var jsonData = pm.response.json();
    var searchInput = pm.request.url.query.get("input");
    if (jsonData.products && jsonData.products.length > 0) {
        var product = jsonData.products[0];
        var nameMatch = product.name.toLowerCase().includes(searchInput.toLowerCase());
        var descMatch = product.description && product.description.toLowerCase().includes(searchInput.toLowerCase());
        pm.expect(nameMatch || descMatch).to.be.true;
    }
});
```

---

#### 3.5. Update Product - Scripts de Test

**Pre-request Script** :
```javascript
// Vérifier si le token existe
if (!pm.environment.get("token")) {
    pm.test.skip("Token not found - skipping test");
    console.log("Warning: No token found. Please login first.");
}

// Vérifier si productId existe
if (!pm.environment.get("productId")) {
    console.log("Warning: No productId found. Using default value 1");
    pm.environment.set("productId", "1");
}
```

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Product update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product Updated successfully");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});

// Test 5: Vérifier que le produit a été mis à jour
pm.test("Product was updated successfully", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    console.log("Product updated successfully");
});
```

---

#### 3.6. Delete Product - Scripts de Test

**Pre-request Script** :
```javascript
// Vérifier si le token existe
if (!pm.environment.get("token")) {
    pm.test.skip("Token not found - skipping test");
    console.log("Warning: No token found. Please login first.");
}

// Vérifier si productId existe
if (!pm.environment.get("productId")) {
    console.log("Warning: No productId found. Using default value 1");
    pm.environment.set("productId", "1");
}
```

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Product delete message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product Deleted successfully");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});

// Test 5: Vérifier que le produit est supprimé
pm.test("Product should be deleted", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    console.log("Product deleted successfully");
});
```

---

#### 4.1. Add Supplier - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Supplier creation message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Supplier Saved Successfully");
});

// Test 4: Sauvegarder l'ID du fournisseur créé
pm.test("Save supplier ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.supplier && jsonData.supplier.id) {
        pm.environment.set("supplierId", jsonData.supplier.id);
        console.log("Supplier ID saved:", jsonData.supplier.id);
    }
});
```

---

#### 5.1. Purchase Transaction - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Purchase message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Purchase Made successfully");
});

// Test 4: Sauvegarder l'ID de la transaction créée
pm.test("Save transaction ID if returned", function () {
    var jsonData = pm.response.json();
    if (jsonData.transaction && jsonData.transaction.id) {
        pm.environment.set("transactionId", jsonData.transaction.id);
        console.log("Transaction ID saved:", jsonData.transaction.id);
    }
});
```

---

#### 5.2. Sell Transaction - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Sell message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Product Sale successfully made");
});
```

---

#### 5.3. Get All Transactions - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains transactions array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('transactions');
    pm.expect(jsonData.transactions).to.be.an('array');
});

// Test 4: Vérifier la pagination
pm.test("Response contains pagination info", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('totalPages');
    pm.expect(jsonData).to.have.property('totalElements');
    pm.expect(jsonData.totalPages).to.be.a('number');
    pm.expect(jsonData.totalElements).to.be.a('number');
});

// Test 5: Vérifier la structure des transactions
pm.test("Transactions have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.transactions && jsonData.transactions.length > 0) {
        var transaction = jsonData.transactions[0];
        pm.expect(transaction).to.have.property('id');
        pm.expect(transaction).to.have.property('transactionType');
        pm.expect(transaction).to.have.property('status');
        pm.expect(transaction).to.have.property('totalProducts');
        pm.expect(transaction).to.have.property('totalPrice');
    }
});
```

---

#### 5.4. Get Transaction By ID - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains transaction object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('transaction');
    pm.expect(jsonData.transaction).to.be.an('object');
});

// Test 4: Vérifier les champs de la transaction
pm.test("Transaction has required fields", function () {
    var jsonData = pm.response.json();
    var transaction = jsonData.transaction;
    pm.expect(transaction).to.have.property('id');
    pm.expect(transaction).to.have.property('transactionType');
    pm.expect(transaction).to.have.property('status');
    pm.expect(transaction).to.have.property('totalProducts');
    pm.expect(transaction).to.have.property('totalPrice');
});
```

---

#### 5.5. Get Transactions By Month and Year - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains transactions array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('transactions');
    pm.expect(jsonData.transactions).to.be.an('array');
});

// Test 4: Vérifier que les transactions correspondent au mois/année demandé
pm.test("Transactions match requested month and year", function () {
    var jsonData = pm.response.json();
    var month = parseInt(pm.request.url.query.get("month"));
    var year = parseInt(pm.request.url.query.get("year"));
    
    if (jsonData.transactions && jsonData.transactions.length > 0) {
        var transaction = jsonData.transactions[0];
        if (transaction.createdAt) {
            var transactionDate = new Date(transaction.createdAt);
            pm.expect(transactionDate.getMonth() + 1).to.eql(month);
            pm.expect(transactionDate.getFullYear()).to.eql(year);
        }
    }
});
```

---

#### 5.6. Update Transaction Status - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("Transaction status update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("Transaction Status Successfully Updated");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

#### 6.1. Get All Users - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains users array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('users');
    pm.expect(jsonData.users).to.be.an('array');
});

// Test 4: Vérifier la structure des utilisateurs
pm.test("Users have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.users && jsonData.users.length > 0) {
        var user = jsonData.users[0];
        pm.expect(user).to.have.property('id');
        pm.expect(user).to.have.property('name');
        pm.expect(user).to.have.property('email');
        pm.expect(user).to.have.property('role');
    }
});
```

---

#### 6.2. Get User By ID - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains user object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('user');
    pm.expect(jsonData.user).to.be.an('object');
});

// Test 4: Vérifier les champs de l'utilisateur
pm.test("User has required fields", function () {
    var jsonData = pm.response.json();
    var user = jsonData.user;
    pm.expect(user).to.have.property('id');
    pm.expect(user).to.have.property('name');
    pm.expect(user).to.have.property('email');
    pm.expect(user).to.have.property('role');
});

// Test 5: Vérifier que l'ID correspond
pm.test("User ID matches request", function () {
    var jsonData = pm.response.json();
    var urlParts = pm.request.url.toString().split('/');
    var requestedId = urlParts[urlParts.length - 1];
    pm.expect(jsonData.user.id.toString()).to.eql(requestedId);
});
```

---

#### 6.3. Update User - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("User update message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("User successfully updated");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

#### 6.4. Delete User - Scripts de Test

**Pre-request Script** :
```javascript
// Vérifier si le token existe
if (!pm.environment.get("token")) {
    pm.test.skip("Token not found - skipping test");
    console.log("Warning: No token found. Please login first.");
}
```

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier le message de succès
pm.test("User delete message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
    pm.expect(jsonData.message).to.eql("User successfully Deleted");
});

// Test 4: Vérifier la structure de la réponse
pm.test("Response has status and message", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
});
```

---

#### 6.5. Get User Transactions - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains user and transactions", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('user');
    pm.expect(jsonData.user).to.have.property('transactions');
    pm.expect(jsonData.user.transactions).to.be.an('array');
});

// Test 4: Vérifier les champs de l'utilisateur
pm.test("User has required fields", function () {
    var jsonData = pm.response.json();
    var user = jsonData.user;
    pm.expect(user).to.have.property('id');
    pm.expect(user).to.have.property('name');
    pm.expect(user).to.have.property('email');
});

// Test 5: Vérifier la structure des transactions
pm.test("Transactions have required fields", function () {
    var jsonData = pm.response.json();
    if (jsonData.user.transactions && jsonData.user.transactions.length > 0) {
        var transaction = jsonData.user.transactions[0];
        pm.expect(transaction).to.have.property('id');
        pm.expect(transaction).to.have.property('transactionType');
        pm.expect(transaction).to.have.property('status');
    }
});
```

---

#### 6.6. Get Current User - Scripts de Test

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Vérifier le format JSON
pm.test("Response is JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Vérifier la structure de la réponse
pm.test("Response contains user object", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('id');
    pm.expect(jsonData).to.have.property('email');
    pm.expect(jsonData).to.have.property('name');
});

// Test 4: Vérifier que l'email correspond au token
pm.test("User email matches token", function () {
    var jsonData = pm.response.json();
    // L'email devrait correspondre à celui utilisé pour le login
    pm.expect(jsonData.email).to.be.a('string');
    pm.expect(jsonData.email).to.include('@');
});

// Test 5: Vérifier les champs requis
pm.test("User has required fields", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('role');
    pm.expect(jsonData).to.have.property('phoneNumber');
});
```

---

### Scripts Postman Globaux

#### Collection Pre-request Script

Ajoutez ce script dans la section "Pre-request Script" de votre collection :

```javascript
// Vérifier si le token existe pour les requêtes authentifiées
var requiresAuth = pm.request.headers.has("Authorization");
if (requiresAuth && !pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
    // Optionnel: arrêter la requête
    // throw new Error("Token required but not found");
}
```

#### Collection Test Script

Ajoutez ce script dans la section "Tests" de votre collection :

```javascript
// Tests globaux pour toutes les requêtes

// Test 1: Vérifier que la réponse n'est pas vide
pm.test("Response is not empty", function () {
    pm.expect(pm.response.text()).to.not.be.empty;
});

// Test 2: Vérifier le temps de réponse (moins de 5 secondes)
pm.test("Response time is acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(5000);
});

// Test 3: Gérer les erreurs serveur avec des messages utiles
pm.test("Check for server errors", function () {
    var statusCode = pm.response.code;
    
    if (statusCode === 500) {
        var jsonData = pm.response.json();
        console.log("❌ Server Error (500):");
        console.log("Message:", jsonData.message);
        console.log("Possible causes:");
        console.log("- Database constraint violation");
        console.log("- Missing required data (category, supplier, etc.)");
        console.log("- Invalid data format");
        console.log("- Check backend logs for more details");
        // Ne pas faire échouer le test, juste logger l'erreur
    } else if (statusCode === 401) {
        var jsonData = pm.response.json();
        console.log("❌ Unauthorized (401):");
        console.log("Message:", jsonData.message);
        console.log("Solution: Login first to get a valid token");
    } else if (statusCode === 403) {
        var jsonData = pm.response.json();
        console.log("❌ Forbidden (403):");
        console.log("Message:", jsonData.message);
        console.log("Solution: You need ADMIN role for this endpoint");
    } else if ([500, 502, 503, 504].includes(statusCode)) {
        // Pour les autres erreurs serveur, faire échouer le test
        pm.expect(statusCode).to.not.be.oneOf([500, 502, 503, 504]);
    }
});
```

---

### Scripts pour Tests d'Erreur

#### Test 404 Not Found

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP 404
pm.test("Status code is 404", function () {
    pm.response.to.have.status(404);
});

// Test 2: Vérifier le message d'erreur
pm.test("Error message is correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData.status).to.eql(404);
});
```

#### Test 400 Bad Request

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP 400
pm.test("Status code is 400", function () {
    pm.response.to.have.status(400);
});

// Test 2: Vérifier le message d'erreur
pm.test("Error message is present", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData.status).to.eql(400);
});
```

#### Test 401 Unauthorized

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP 401
pm.test("Status code is 401", function () {
    pm.response.to.have.status(401);
});

// Test 2: Vérifier le message d'erreur
pm.test("Unauthorized message is present", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData.status).to.eql(401);
});
```

#### Test 403 Forbidden

**Tests Tab** :
```javascript
// Test 1: Vérifier le statut HTTP 403
pm.test("Status code is 403", function () {
    pm.response.to.have.status(403);
});

// Test 2: Vérifier le message d'erreur
pm.test("Forbidden message is present", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData.status).to.eql(403);
});
```

---

### Scripts Utiles pour Automatisation

#### Script pour Exécuter une Suite de Tests

Créez une requête "Test Suite Runner" avec ce script :

```javascript
// Script pour exécuter une suite de tests dans l'ordre
var testOrder = [
    "Register User",
    "Login User",
    "Create Category",
    "Get All Categories",
    "Add Supplier",
    "Create Product",
    "Purchase Transaction"
];

// Sauvegarder l'ordre des tests
pm.environment.set("testOrder", JSON.stringify(testOrder));
console.log("Test suite order saved");
```

#### Script pour Nettoyer les Données de Test

**Tests Tab** :
```javascript
// Nettoyer les variables d'environnement après les tests
pm.test("Cleanup test data", function () {
    // Optionnel: supprimer les IDs créés pendant les tests
    // pm.environment.unset("categoryId");
    // pm.environment.unset("productId");
    // pm.environment.unset("supplierId");
    console.log("Test cleanup completed");
});
```

---

### Exécution Automatique avec Newman

Pour exécuter automatiquement tous les tests avec Newman (CLI de Postman) :

```bash
# Installer Newman
npm install -g newman

# Exécuter la collection
newman run Collection.json -e Environment.json --reporters cli,html

# Avec rapport HTML
newman run Collection.json -e Environment.json --reporters html --reporter-html-export report.html
```

---

## Notes Importantes pour l'Automatisation

1. **Ordre d'exécution** : Exécutez toujours Register → Login avant les autres tests
2. **Variables d'environnement** : Les IDs sont sauvegardés automatiquement pour les tests suivants
3. **Token JWT** : Le token est sauvegardé automatiquement après le login
4. **Nettoyage** : Nettoyez les données de test après chaque exécution si nécessaire
5. **Assertions** : Tous les tests vérifient le statut HTTP, le format JSON et la structure des données

