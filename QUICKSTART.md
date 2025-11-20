# Guide de Démarrage Rapide - Fonctionnalité Formule

Ce guide vous aidera à démarrer rapidement avec l'implémentation de la fonctionnalité formule.

## Prérequis

- Java JDK 11 ou supérieur
- JavaFX SDK
- IDE avec support Maven/Gradle (IntelliJ IDEA, Eclipse, NetBeans)
- Base de données (MySQL, PostgreSQL, ou H2 pour le développement)

## Installation Rapide

### 1. Créer le Projet Maven

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.inventaire</groupId>
    <artifactId>inventaire-formule</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <javafx.version>17.0.2</javafx.version>
    </properties>
    
    <dependencies>
        <!-- JavaFX -->
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-controls</artifactId>
            <version>${javafx.version}</version>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-fxml</artifactId>
            <version>${javafx.version}</version>
        </dependency>
        
        <!-- Base de données -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.1.214</version>
        </dependency>
        
        <!-- Tests -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.9.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.openjfx</groupId>
                <artifactId>javafx-maven-plugin</artifactId>
                <version>0.0.8</version>
                <configuration>
                    <mainClass>com.inventaire.Main</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2. Structure de Dossiers

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── inventaire/
│   │           ├── Main.java
│   │           ├── model/
│   │           │   ├── Equipement.java
│   │           │   ├── Formula.java
│   │           │   └── CalculResult.java
│   │           ├── formulas/
│   │           │   ├── CoutTotalFormula.java
│   │           │   ├── AmortissementLineaireFormula.java
│   │           │   └── ValeurResiduelleFormula.java
│   │           ├── service/
│   │           │   └── FormuleManager.java
│   │           └── controller/
│   │               └── FormuleController.java
│   └── resources/
│       ├── fxml/
│       │   └── formule_view.fxml
│       └── css/
│           └── formule_style.css
└── test/
    └── java/
        └── com/
            └── inventaire/
                └── FormulaTest.java
```

### 3. Classe Main

```java
package com.inventaire;

import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.scene.Scene;
import javafx.scene.Parent;
import javafx.stage.Stage;

public class Main extends Application {
    
    @Override
    public void start(Stage primaryStage) throws Exception {
        FXMLLoader loader = new FXMLLoader(
            getClass().getResource("/fxml/formule_view.fxml")
        );
        Parent root = loader.load();
        
        Scene scene = new Scene(root, 1000, 700);
        
        primaryStage.setTitle("Inventaire - Calculateur de Formules");
        primaryStage.setScene(scene);
        primaryStage.show();
    }
    
    public static void main(String[] args) {
        launch(args);
    }
}
```

## Première Formule en 5 Minutes

### Étape 1: Créer l'Interface Formula

```java
package com.inventaire.model;

import java.util.List;
import java.util.Map;

public interface Formula {
    String getName();
    String getDescription();
    List<String> getRequiredParameters();
    String getResultUnit();
    double calculate(Map<String, Double> parameters);
    boolean validateParameters(Map<String, Double> parameters);
}
```

### Étape 2: Implémenter une Formule Simple

```java
package com.inventaire.formulas;

import com.inventaire.model.Formula;
import java.util.*;

public class CoutTotalFormula implements Formula {
    
    @Override
    public String getName() {
        return "Coût Total";
    }
    
    @Override
    public String getDescription() {
        return "Calcule le coût total d'un équipement";
    }
    
    @Override
    public List<String> getRequiredParameters() {
        return Arrays.asList("prixAchat", "fraisInstallation");
    }
    
    @Override
    public String getResultUnit() {
        return "€";
    }
    
    @Override
    public boolean validateParameters(Map<String, Double> parameters) {
        if (parameters == null) return false;
        for (String param : getRequiredParameters()) {
            if (!parameters.containsKey(param) || parameters.get(param) < 0) {
                return false;
            }
        }
        return true;
    }
    
    @Override
    public double calculate(Map<String, Double> parameters) {
        if (!validateParameters(parameters)) {
            throw new IllegalArgumentException("Paramètres invalides");
        }
        return parameters.get("prixAchat") + parameters.get("fraisInstallation");
    }
}
```

### Étape 3: Créer le Manager

```java
package com.inventaire.service;

import com.inventaire.model.Formula;
import com.inventaire.formulas.CoutTotalFormula;
import java.util.*;

public class FormuleManager {
    private Map<String, Formula> formulas = new HashMap<>();
    
    public FormuleManager() {
        registerFormula(new CoutTotalFormula());
        // Ajouter d'autres formules ici
    }
    
    public void registerFormula(Formula formula) {
        formulas.put(formula.getName(), formula);
    }
    
    public double calculate(String formulaName, Map<String, Double> parameters) {
        Formula formula = formulas.get(formulaName);
        if (formula == null) {
            throw new IllegalArgumentException("Formule inconnue: " + formulaName);
        }
        return formula.calculate(parameters);
    }
    
    public List<Formula> getAvailableFormulas() {
        return new ArrayList<>(formulas.values());
    }
}
```

### Étape 4: Tester

```java
package com.inventaire;

import com.inventaire.service.FormuleManager;
import java.util.HashMap;
import java.util.Map;

public class Test {
    public static void main(String[] args) {
        FormuleManager manager = new FormuleManager();
        
        Map<String, Double> params = new HashMap<>();
        params.put("prixAchat", 1500.0);
        params.put("fraisInstallation", 200.0);
        
        double result = manager.calculate("Coût Total", params);
        System.out.println("Coût Total: " + result + "€");
        // Affiche: Coût Total: 1700.0€
    }
}
```

## Commandes Utiles

### Compiler et Exécuter

```bash
# Avec Maven
mvn clean compile
mvn javafx:run

# Avec Gradle
gradle clean build
gradle run
```

### Lancer les Tests

```bash
# Maven
mvn test

# Gradle
gradle test
```

### Créer un JAR Exécutable

```bash
# Maven
mvn clean package

# Gradle
gradle jar
```

## Prochaines Étapes

1. **Ajouter plus de formules** : Consultez [IMPLEMENTATION_EXAMPLE.md](IMPLEMENTATION_EXAMPLE.md) pour des exemples complets

2. **Créer l'interface utilisateur** : Utilisez le fichier FXML fourni dans [IMPLEMENTATION_EXAMPLE.md](IMPLEMENTATION_EXAMPLE.md)

3. **Ajouter la persistance** : Implémentez la sauvegarde des calculs en base de données

4. **Créer des rapports** : Ajoutez l'export PDF/Excel des résultats

5. **Formules personnalisées** : Permettez aux utilisateurs de créer leurs propres formules

## Ressources

- [Spécification complète](FORMULE_FEATURE.md) - Toutes les formules et fonctionnalités
- [Exemples d'implémentation](IMPLEMENTATION_EXAMPLE.md) - Code Java complet
- [JavaFX Documentation](https://openjfx.io/) - Documentation officielle
- [JavaFX Tutorial](https://docs.oracle.com/javafx/) - Tutoriels Oracle

## Support

Pour toute question ou problème :
1. Consultez la documentation complète
2. Vérifiez les exemples de code fournis
3. Créez une issue sur le dépôt GitHub

## Exemple Complet Minimal

Voici un exemple complet qui fonctionne sans interface graphique :

```java
package com.inventaire;

import java.util.*;

// Interface
interface Formula {
    String getName();
    double calculate(Map<String, Double> params);
}

// Implémentation
class SimpleFormula implements Formula {
    public String getName() { return "Coût Total"; }
    public double calculate(Map<String, Double> p) {
        return p.get("prix") + p.get("frais");
    }
}

// Programme principal
public class Demo {
    public static void main(String[] args) {
        Formula formula = new SimpleFormula();
        
        Map<String, Double> params = new HashMap<>();
        params.put("prix", 1000.0);
        params.put("frais", 150.0);
        
        double result = formula.calculate(params);
        System.out.println(formula.getName() + ": " + result + "€");
    }
}
```

Compilez et exécutez :
```bash
javac Demo.java
java Demo
# Output: Coût Total: 1150.0€
```

---

**Félicitations !** Vous avez maintenant une base solide pour implémenter la fonctionnalité formule. Continuez avec les documents détaillés pour une implémentation complète.
