# Fonctionnalité Formule - Spécification

## Vue d'ensemble

Cette fonctionnalité ajoute un système de formules au système d'inventaire de matériel informatique pour automatiser les calculs et analyses.

## Objectifs

- Permettre le calcul automatique de valeurs basées sur des formules prédéfinies
- Faciliter l'analyse et la gestion du parc informatique
- Automatiser les calculs de coûts, d'amortissement et de valorisation

## Types de formules proposées

### 1. Formules de Coût

#### Coût Total
```
Coût Total = Prix d'Achat + Frais d'Installation + Frais de Maintenance
```

#### Coût par Utilisateur
```
Coût par Utilisateur = Coût Total / Nombre d'Utilisateurs
```

#### Coût de Possession (TCO)
```
TCO = Prix d'Achat + (Maintenance Annuelle × Durée de Vie) + Frais de Remplacement
```

### 2. Formules d'Amortissement

#### Amortissement Linéaire
```
Amortissement Annuel = (Prix d'Achat - Valeur Résiduelle) / Durée de Vie
Valeur Actuelle = Prix d'Achat - (Amortissement Annuel × Années d'Utilisation)
```

#### Amortissement Dégressif
```
Taux Dégressif = (100% / Durée de Vie) × Coefficient
Amortissement Année N = Valeur Comptable × Taux Dégressif
```

### 3. Formules de Valorisation

#### Valeur Résiduelle
```
Valeur Résiduelle = Prix d'Achat × (1 - Taux de Dépréciation)^Années
```

#### Pourcentage de Dépréciation
```
Dépréciation (%) = ((Prix d'Achat - Valeur Actuelle) / Prix d'Achat) × 100
```

### 4. Formules d'Inventaire

#### Taux de Renouvellement
```
Taux de Renouvellement (%) = (Équipements Remplacés / Total Équipements) × 100
```

#### Âge Moyen du Parc
```
Âge Moyen = Σ(Âge de chaque équipement) / Nombre Total d'Équipements
```

#### Valeur Totale du Parc
```
Valeur Totale = Σ(Valeur Actuelle de chaque équipement)
```

### 5. Formules de Performance

#### Ratio Coût/Performance
```
Ratio = Coût d'Acquisition / Score de Performance
```

#### Efficacité Énergétique
```
Coût Énergétique Annuel = Consommation (kWh) × Prix kWh × Heures d'Utilisation
```

## Interface Utilisateur Proposée

### Vue Formules
- **Liste des formules disponibles** : Affichage de toutes les formules configurées
- **Sélection de formule** : Menu déroulant ou liste cliquable
- **Paramètres de formule** : Champs pour saisir les variables nécessaires
- **Calcul en temps réel** : Affichage instantané des résultats
- **Historique** : Conservation des calculs précédents

### Fonctionnalités Additionnelles

#### Formules Personnalisées
- Créer des formules personnalisées avec l'éditeur
- Sauvegarder et réutiliser les formules
- Partager les formules entre utilisateurs

#### Export et Rapports
- Exporter les résultats en PDF/Excel
- Générer des rapports automatiques
- Visualisation graphique des tendances

#### Alertes et Notifications
- Alertes basées sur les résultats de formules
- Notifications pour équipements nécessitant un remplacement
- Seuils personnalisables

## Architecture Technique

### Composants JavaFX

```java
// Classe principale pour la gestion des formules
public class FormuleManager {
    private Map<String, Formula> formulas;
    
    public double calculate(String formulaName, Map<String, Double> parameters);
    public void addCustomFormula(String name, String expression);
    public List<Formula> getAvailableFormulas();
}

// Interface pour les formules
public interface Formula {
    String getName();
    String getDescription();
    List<String> getRequiredParameters();
    double calculate(Map<String, Double> values);
}

// Exemple d'implémentation
public class CoutTotalFormula implements Formula {
    public double calculate(Map<String, Double> values) {
        return values.get("prixAchat") + 
               values.get("fraisInstallation") + 
               values.get("fraisMaintenance");
    }
}
```

### Structure FXML

```xml
<VBox xmlns:fx="http://javafx.com/fxml">
    <Label text="Calculateur de Formules"/>
    
    <ComboBox fx:id="formulaSelector">
        <items>
            <FXCollections fx:factory="observableArrayList">
                <String fx:value="Coût Total"/>
                <String fx:value="Amortissement Linéaire"/>
                <String fx:value="Valeur Résiduelle"/>
            </FXCollections>
        </items>
    </ComboBox>
    
    <GridPane fx:id="parametersGrid">
        <!-- Champs dynamiques selon la formule sélectionnée -->
    </GridPane>
    
    <Button text="Calculer" onAction="#calculateFormula"/>
    
    <Label fx:id="resultLabel" styleClass="result"/>
</VBox>
```

### Contrôleur

```java
public class FormuleController {
    @FXML private ComboBox<String> formulaSelector;
    @FXML private GridPane parametersGrid;
    @FXML private Label resultLabel;
    
    private FormuleManager formuleManager;
    
    @FXML
    public void initialize() {
        formuleManager = new FormuleManager();
        loadFormulas();
        setupListeners();
    }
    
    @FXML
    private void calculateFormula() {
        String selectedFormula = formulaSelector.getValue();
        Map<String, Double> parameters = collectParameters();
        double result = formuleManager.calculate(selectedFormula, parameters);
        displayResult(result);
    }
    
    private void displayResult(double result) {
        resultLabel.setText(String.format("Résultat: %.2f €", result));
    }
}
```

## Base de Données

### Table Formules
```sql
CREATE TABLE formules (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100) NOT NULL,
    description TEXT,
    expression TEXT NOT NULL,
    type VARCHAR(50),
    date_creation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    utilisateur_id INT,
    est_active BOOLEAN DEFAULT TRUE
);
```

### Table Calculs
```sql
CREATE TABLE calculs_historique (
    id INT PRIMARY KEY AUTO_INCREMENT,
    formule_id INT,
    equipement_id INT,
    parametres JSON,
    resultat DECIMAL(10, 2),
    date_calcul TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (formule_id) REFERENCES formules(id),
    FOREIGN KEY (equipement_id) REFERENCES equipements(id)
);
```

## Cas d'Utilisation

### Scénario 1: Calcul de l'Amortissement
1. L'utilisateur sélectionne "Amortissement Linéaire"
2. Il saisit le prix d'achat (1500€), la durée de vie (5 ans)
3. Le système calcule automatiquement l'amortissement annuel (300€)
4. La valeur actuelle est affichée selon l'âge de l'équipement

### Scénario 2: Analyse du Parc
1. L'utilisateur lance "Valeur Totale du Parc"
2. Le système parcourt tous les équipements
3. Applique la formule de dépréciation à chacun
4. Affiche la valeur totale actualisée

### Scénario 3: Formule Personnalisée
1. L'utilisateur crée une formule "Coût par mois d'utilisation"
2. Définit l'expression: (Prix d'Achat + Maintenance) / (Durée de Vie × 12)
3. Sauvegarde la formule
4. L'applique à un ou plusieurs équipements

## Tests et Validation

### Tests Unitaires
- Vérifier la précision des calculs
- Valider la gestion des erreurs (division par zéro, valeurs négatives)
- Tester les formules avec différents jeux de données

### Tests d'Intégration
- Vérifier l'interaction avec la base de données
- Tester l'export des résultats
- Valider l'interface utilisateur

### Tests de Performance
- Calcul sur un grand nombre d'équipements
- Temps de réponse pour les formules complexes

## Feuille de Route

### Phase 1: MVP (Minimum Viable Product)
- Implémentation des 5 formules de base
- Interface simple de sélection et calcul
- Affichage des résultats

### Phase 2: Fonctionnalités Avancées
- Formules personnalisées
- Historique des calculs
- Export des résultats

### Phase 3: Optimisations
- Calculs en arrière-plan
- Mise en cache des résultats
- Visualisations graphiques

## Considérations de Sécurité

- Validation des entrées utilisateur
- Protection contre l'injection de code dans les formules personnalisées
- Limitation des droits d'accès aux formules sensibles
- Audit trail pour les modifications de formules

## Documentation Utilisateur

- Guide d'utilisation des formules prédéfinies
- Tutoriel pour créer des formules personnalisées
- FAQ sur les calculs d'amortissement et de valorisation
- Exemples concrets d'utilisation

## Maintenance et Support

- Mise à jour régulière des formules selon la législation
- Support pour nouveaux types de calculs
- Correction des bugs signalés
- Amélioration continue basée sur les retours utilisateurs
