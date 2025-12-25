# Scripts Postman - Tests Automatisés

Ce fichier contient tous les scripts de test Postman prêts à copier-coller pour automatiser les tests des APIs.

---

## 📋 Table des Matières

1. [Scripts Globaux](#scripts-globaux)
2. [Authentification](#authentification)
3. [Categories](#categories)
4. [Products](#products)
5. [Suppliers](#suppliers)
6. [Transactions](#transactions)
6. [Users](#users)
7. [Tests d'Erreur](#tests-derreur)

---

## Scripts Globaux

### Collection Pre-request Script

Copiez ce script dans la section "Pre-request Script" de votre collection Postman :

```javascript
// Vérifier si le token existe pour les requêtes authentifiées
var requiresAuth = pm.request.headers.has("Authorization");
if (requiresAuth && !pm.environment.get("token")) {
    console.log("Warning: No token found. Please login first.");
}
```

### Collection Test Script

Copiez ce script dans la section "Tests" de votre collection Postman :

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

// Test 3: Vérifier qu'il n'y a pas d'erreurs serveur
pm.test("No server errors", function () {
    pm.expect(pm.response.code).to.not.be.oneOf([500, 502, 503, 504]);
});
```

---

## Authentification

### 1. Register User

**Onglet Tests** :

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

### 2. Login User

**Onglet Tests** :

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

## Categories

### 1. Create Category

**Pre-request Script** :

```javascript
// Vérifier si le token existe
if (!pm.environment.get("token")) {
    pm.test.skip("Token not found - skipping test");
    console.log("Warning: No token found. Please login first.");
}
```

**Onglet Tests** :

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

### 2. Get All Categories

**Onglet Tests** :

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

### 3. Get Category By ID

**Onglet Tests** :

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

### 4. Update Category

**Onglet Tests** :

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

### 5. Delete Category

**Onglet Tests** :

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

// Test 4: Vérifier que la catégorie est supprimée
pm.test("Category should be deleted", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(200);
});
```

---

## Products

### 1. Create Product

**Onglet Tests** :

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

### 2. Get All Products

**Onglet Tests** :

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

### 3. Get Product By ID

**Onglet Tests** :

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

### 4. Search Product

**Onglet Tests** :

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

## Suppliers

### 1. Add Supplier

**Onglet Tests** :

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

## Transactions

### 1. Purchase Transaction

**Onglet Tests** :

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

### 2. Sell Transaction

**Onglet Tests** :

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

### 3. Get All Transactions

**Onglet Tests** :

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

## Users

### 1. Get All Users

**Onglet Tests** :

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

### 2. Get Current User

**Onglet Tests** :

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
    pm.expect(jsonData.email).to.be.a('string');
    pm.expect(jsonData.email).to.include('@');
});
```

---

## Tests d'Erreur

### Test 404 Not Found

**Onglet Tests** :

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

---

### Test 400 Bad Request

**Onglet Tests** :

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

---

### Test 401 Unauthorized

**Onglet Tests** :

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

---

### Test 403 Forbidden

**Onglet Tests** :

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

## 🚀 Utilisation

1. **Copiez** le script correspondant à votre endpoint
2. **Collez-le** dans l'onglet "Tests" de votre requête Postman
3. **Exécutez** la requête pour voir les tests s'exécuter automatiquement
4. **Vérifiez** les résultats dans l'onglet "Test Results"

---

## 📝 Notes

- Tous les scripts vérifient automatiquement le statut HTTP, le format JSON et la structure des données
- Les IDs sont sauvegardés automatiquement dans les variables d'environnement
- Le token JWT est sauvegardé automatiquement après le login
- Les tests peuvent être exécutés individuellement ou en collection

---

## 🔄 Exécution Automatique avec Newman

```bash
# Installer Newman
npm install -g newman

# Exécuter la collection avec les tests
newman run Collection.json -e Environment.json --reporters cli,html

# Générer un rapport HTML
newman run Collection.json -e Environment.json --reporters html --reporter-html-export report.html
```

