# Exemple d'Implémentation - Fonctionnalité Formule

Ce document fournit des exemples concrets d'implémentation de la fonctionnalité formule pour le système d'inventaire.

## Structure du Projet

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── inventaire/
│   │           ├── model/
│   │           │   ├── Equipement.java
│   │           │   ├── Formula.java
│   │           │   └── CalculResult.java
│   │           ├── service/
│   │           │   ├── FormuleManager.java
│   │           │   └── CalculationService.java
│   │           ├── formulas/
│   │           │   ├── CoutTotalFormula.java
│   │           │   ├── AmortissementLineaireFormula.java
│   │           │   └── ValeurResiduelleFormula.java
│   │           └── controller/
│   │               └── FormuleController.java
│   └── resources/
│       └── fxml/
│           └── formule_view.fxml
```

## Classes Modèle

### Equipement.java
```java
package com.inventaire.model;

import java.time.LocalDate;

public class Equipement {
    private int id;
    private String nom;
    private String type;
    private double prixAchat;
    private LocalDate dateAchat;
    private int dureeVieAnnees;
    private double valeurResiduelle;
    private String statut;
    
    // Constructeurs
    public Equipement() {}
    
    public Equipement(String nom, String type, double prixAchat, 
                      LocalDate dateAchat, int dureeVieAnnees) {
        this.nom = nom;
        this.type = type;
        this.prixAchat = prixAchat;
        this.dateAchat = dateAchat;
        this.dureeVieAnnees = dureeVieAnnees;
        this.valeurResiduelle = 0.0;
        this.statut = "Actif";
    }
    
    // Getters et Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getNom() { return nom; }
    public void setNom(String nom) { this.nom = nom; }
    
    public String getType() { return type; }
    public void setType(String type) { this.type = type; }
    
    public double getPrixAchat() { return prixAchat; }
    public void setPrixAchat(double prixAchat) { this.prixAchat = prixAchat; }
    
    public LocalDate getDateAchat() { return dateAchat; }
    public void setDateAchat(LocalDate dateAchat) { this.dateAchat = dateAchat; }
    
    public int getDureeVieAnnees() { return dureeVieAnnees; }
    public void setDureeVieAnnees(int dureeVieAnnees) { 
        this.dureeVieAnnees = dureeVieAnnees; 
    }
    
    public double getValeurResiduelle() { return valeurResiduelle; }
    public void setValeurResiduelle(double valeurResiduelle) { 
        this.valeurResiduelle = valeurResiduelle; 
    }
    
    public String getStatut() { return statut; }
    public void setStatut(String statut) { this.statut = statut; }
    
    // Méthodes utilitaires
    public int getAgeEnAnnees() {
        return LocalDate.now().getYear() - dateAchat.getYear();
    }
    
    @Override
    public String toString() {
        return String.format("%s (%s) - %.2f€", nom, type, prixAchat);
    }
}
```

### Formula.java
```java
package com.inventaire.model;

import java.util.List;
import java.util.Map;

public interface Formula {
    /**
     * Retourne le nom de la formule
     */
    String getName();
    
    /**
     * Retourne la description de la formule
     */
    String getDescription();
    
    /**
     * Retourne la liste des paramètres requis
     */
    List<String> getRequiredParameters();
    
    /**
     * Retourne l'unité du résultat (€, %, années, etc.)
     */
    String getResultUnit();
    
    /**
     * Calcule le résultat de la formule
     * @param parameters Map contenant les paramètres avec leurs valeurs
     * @return Le résultat du calcul
     * @throws IllegalArgumentException si les paramètres sont invalides
     */
    double calculate(Map<String, Double> parameters);
    
    /**
     * Valide les paramètres avant le calcul
     */
    boolean validateParameters(Map<String, Double> parameters);
}
```

### CalculResult.java
```java
package com.inventaire.model;

import java.time.LocalDateTime;

public class CalculResult {
    private String formulaName;
    private double result;
    private String unit;
    private LocalDateTime calculationDate;
    private Map<String, Double> parameters;
    
    public CalculResult(String formulaName, double result, String unit,
                       Map<String, Double> parameters) {
        this.formulaName = formulaName;
        this.result = result;
        this.unit = unit;
        this.parameters = parameters;
        this.calculationDate = LocalDateTime.now();
    }
    
    // Getters
    public String getFormulaName() { return formulaName; }
    public double getResult() { return result; }
    public String getUnit() { return unit; }
    public LocalDateTime getCalculationDate() { return calculationDate; }
    public Map<String, Double> getParameters() { return parameters; }
    
    public String getFormattedResult() {
        return String.format("%.2f %s", result, unit);
    }
    
    @Override
    public String toString() {
        return String.format("%s: %.2f %s (calculé le %s)", 
            formulaName, result, unit, calculationDate);
    }
}
```

## Implémentations de Formules

### CoutTotalFormula.java
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
        return "Calcule le coût total incluant le prix d'achat, " +
               "les frais d'installation et les frais de maintenance";
    }
    
    @Override
    public List<String> getRequiredParameters() {
        return Arrays.asList(
            "prixAchat",
            "fraisInstallation",
            "fraisMaintenance"
        );
    }
    
    @Override
    public String getResultUnit() {
        return "€";
    }
    
    @Override
    public boolean validateParameters(Map<String, Double> parameters) {
        if (parameters == null) return false;
        
        for (String param : getRequiredParameters()) {
            if (!parameters.containsKey(param)) return false;
            if (parameters.get(param) < 0) return false;
        }
        return true;
    }
    
    @Override
    public double calculate(Map<String, Double> parameters) {
        if (!validateParameters(parameters)) {
            throw new IllegalArgumentException(
                "Paramètres invalides pour le calcul du coût total"
            );
        }
        
        double prixAchat = parameters.get("prixAchat");
        double fraisInstallation = parameters.get("fraisInstallation");
        double fraisMaintenance = parameters.get("fraisMaintenance");
        
        return prixAchat + fraisInstallation + fraisMaintenance;
    }
}
```

### AmortissementLineaireFormula.java
```java
package com.inventaire.formulas;

import com.inventaire.model.Formula;
import java.util.*;

public class AmortissementLineaireFormula implements Formula {
    
    @Override
    public String getName() {
        return "Amortissement Linéaire";
    }
    
    @Override
    public String getDescription() {
        return "Calcule la valeur actuelle après amortissement linéaire";
    }
    
    @Override
    public List<String> getRequiredParameters() {
        return Arrays.asList(
            "prixAchat",
            "dureeVieAnnees",
            "anneesUtilisation",
            "valeurResiduelle"
        );
    }
    
    @Override
    public String getResultUnit() {
        return "€";
    }
    
    @Override
    public boolean validateParameters(Map<String, Double> parameters) {
        if (parameters == null) return false;
        
        for (String param : getRequiredParameters()) {
            if (!parameters.containsKey(param)) return false;
        }
        
        double prixAchat = parameters.get("prixAchat");
        double dureeVie = parameters.get("dureeVieAnnees");
        double anneesUtil = parameters.get("anneesUtilisation");
        double valeurRes = parameters.get("valeurResiduelle");
        
        return prixAchat >= 0 && dureeVie > 0 && 
               anneesUtil >= 0 && anneesUtil <= dureeVie &&
               valeurRes >= 0 && valeurRes <= prixAchat;
    }
    
    @Override
    public double calculate(Map<String, Double> parameters) {
        if (!validateParameters(parameters)) {
            throw new IllegalArgumentException(
                "Paramètres invalides pour l'amortissement linéaire"
            );
        }
        
        double prixAchat = parameters.get("prixAchat");
        double dureeVie = parameters.get("dureeVieAnnees");
        double anneesUtil = parameters.get("anneesUtilisation");
        double valeurRes = parameters.get("valeurResiduelle");
        
        // Formule: Valeur = PrixAchat - ((PrixAchat - ValeurRes) / DureeVie * Années)
        double amortissementAnnuel = (prixAchat - valeurRes) / dureeVie;
        double valeurActuelle = prixAchat - (amortissementAnnuel * anneesUtil);
        
        return Math.max(valeurActuelle, valeurRes);
    }
}
```

### ValeurResiduelleFormula.java
```java
package com.inventaire.formulas;

import com.inventaire.model.Formula;
import java.util.*;

public class ValeurResiduelleFormula implements Formula {
    
    @Override
    public String getName() {
        return "Valeur Résiduelle";
    }
    
    @Override
    public String getDescription() {
        return "Calcule la valeur résiduelle avec dépréciation exponentielle";
    }
    
    @Override
    public List<String> getRequiredParameters() {
        return Arrays.asList(
            "prixAchat",
            "tauxDepreciation",
            "anneesUtilisation"
        );
    }
    
    @Override
    public String getResultUnit() {
        return "€";
    }
    
    @Override
    public boolean validateParameters(Map<String, Double> parameters) {
        if (parameters == null) return false;
        
        for (String param : getRequiredParameters()) {
            if (!parameters.containsKey(param)) return false;
        }
        
        double prixAchat = parameters.get("prixAchat");
        double taux = parameters.get("tauxDepreciation");
        double annees = parameters.get("anneesUtilisation");
        
        return prixAchat >= 0 && taux >= 0 && taux <= 1 && annees >= 0;
    }
    
    @Override
    public double calculate(Map<String, Double> parameters) {
        if (!validateParameters(parameters)) {
            throw new IllegalArgumentException(
                "Paramètres invalides pour la valeur résiduelle"
            );
        }
        
        double prixAchat = parameters.get("prixAchat");
        double taux = parameters.get("tauxDepreciation");
        double annees = parameters.get("anneesUtilisation");
        
        // Formule: Valeur = PrixAchat × (1 - Taux)^Années
        return prixAchat * Math.pow(1 - taux, annees);
    }
}
```

## Service de Gestion

### FormuleManager.java
```java
package com.inventaire.service;

import com.inventaire.model.Formula;
import com.inventaire.model.CalculResult;
import com.inventaire.formulas.*;
import java.util.*;

public class FormuleManager {
    private Map<String, Formula> formulas;
    private List<CalculResult> history;
    
    public FormuleManager() {
        this.formulas = new HashMap<>();
        this.history = new ArrayList<>();
        initializeFormulas();
    }
    
    private void initializeFormulas() {
        registerFormula(new CoutTotalFormula());
        registerFormula(new AmortissementLineaireFormula());
        registerFormula(new ValeurResiduelleFormula());
    }
    
    public void registerFormula(Formula formula) {
        formulas.put(formula.getName(), formula);
    }
    
    public CalculResult calculate(String formulaName, 
                                  Map<String, Double> parameters) {
        Formula formula = formulas.get(formulaName);
        if (formula == null) {
            throw new IllegalArgumentException(
                "Formule inconnue: " + formulaName
            );
        }
        
        double result = formula.calculate(parameters);
        CalculResult calculResult = new CalculResult(
            formulaName, result, formula.getResultUnit(), parameters
        );
        
        history.add(calculResult);
        return calculResult;
    }
    
    public List<Formula> getAvailableFormulas() {
        return new ArrayList<>(formulas.values());
    }
    
    public Formula getFormula(String name) {
        return formulas.get(name);
    }
    
    public List<CalculResult> getHistory() {
        return new ArrayList<>(history);
    }
    
    public void clearHistory() {
        history.clear();
    }
}
```

## Contrôleur JavaFX

### FormuleController.java
```java
package com.inventaire.controller;

import com.inventaire.model.*;
import com.inventaire.service.FormuleManager;
import javafx.fxml.FXML;
import javafx.scene.control.*;
import javafx.scene.layout.GridPane;
import javafx.collections.FXCollections;
import java.util.*;

public class FormuleController {
    
    @FXML private ComboBox<String> formulaSelector;
    @FXML private GridPane parametersGrid;
    @FXML private Button calculateButton;
    @FXML private Label resultLabel;
    @FXML private TextArea descriptionArea;
    @FXML private ListView<String> historyList;
    
    private FormuleManager formuleManager;
    private Map<String, TextField> parameterFields;
    
    @FXML
    public void initialize() {
        formuleManager = new FormuleManager();
        parameterFields = new HashMap<>();
        
        loadFormulas();
        setupListeners();
    }
    
    private void loadFormulas() {
        List<String> formulaNames = new ArrayList<>();
        for (Formula formula : formuleManager.getAvailableFormulas()) {
            formulaNames.add(formula.getName());
        }
        formulaSelector.setItems(FXCollections.observableArrayList(formulaNames));
    }
    
    private void setupListeners() {
        formulaSelector.valueProperty().addListener((obs, oldVal, newVal) -> {
            if (newVal != null) {
                updateParameterFields(newVal);
            }
        });
    }
    
    private void updateParameterFields(String formulaName) {
        Formula formula = formuleManager.getFormula(formulaName);
        if (formula == null) return;
        
        // Mettre à jour la description
        descriptionArea.setText(formula.getDescription());
        
        // Nettoyer les champs existants
        parametersGrid.getChildren().clear();
        parameterFields.clear();
        
        // Créer les champs pour chaque paramètre
        List<String> params = formula.getRequiredParameters();
        for (int i = 0; i < params.size(); i++) {
            String param = params.get(i);
            
            Label label = new Label(formatParameterName(param) + ":");
            TextField textField = new TextField();
            textField.setPromptText("Entrez " + formatParameterName(param));
            
            parametersGrid.add(label, 0, i);
            parametersGrid.add(textField, 1, i);
            
            parameterFields.put(param, textField);
        }
    }
    
    private String formatParameterName(String param) {
        // Convertir camelCase en texte lisible
        return param.replaceAll("([A-Z])", " $1")
                   .toLowerCase()
                   .substring(0, 1).toUpperCase() + 
               param.replaceAll("([A-Z])", " $1")
                   .toLowerCase()
                   .substring(1);
    }
    
    @FXML
    private void calculateFormula() {
        String selectedFormula = formulaSelector.getValue();
        if (selectedFormula == null) {
            showError("Veuillez sélectionner une formule");
            return;
        }
        
        try {
            Map<String, Double> parameters = collectParameters();
            CalculResult result = formuleManager.calculate(
                selectedFormula, parameters
            );
            
            displayResult(result);
            updateHistory();
            
        } catch (NumberFormatException e) {
            showError("Veuillez entrer des valeurs numériques valides");
        } catch (IllegalArgumentException e) {
            showError(e.getMessage());
        }
    }
    
    private Map<String, Double> collectParameters() {
        Map<String, Double> parameters = new HashMap<>();
        
        for (Map.Entry<String, TextField> entry : parameterFields.entrySet()) {
            String paramName = entry.getKey();
            TextField field = entry.getValue();
            
            String valueText = field.getText().trim();
            if (valueText.isEmpty()) {
                throw new IllegalArgumentException(
                    "Le paramètre " + formatParameterName(paramName) + 
                    " est requis"
                );
            }
            
            double value = Double.parseDouble(valueText);
            parameters.put(paramName, value);
        }
        
        return parameters;
    }
    
    private void displayResult(CalculResult result) {
        resultLabel.setText(
            String.format("Résultat: %.2f %s", 
                result.getResult(), 
                result.getUnit()
            )
        );
        resultLabel.setStyle("-fx-font-size: 18px; -fx-font-weight: bold;");
    }
    
    private void updateHistory() {
        List<String> historyItems = new ArrayList<>();
        for (CalculResult result : formuleManager.getHistory()) {
            historyItems.add(result.toString());
        }
        historyList.setItems(FXCollections.observableArrayList(historyItems));
    }
    
    private void showError(String message) {
        Alert alert = new Alert(Alert.AlertType.ERROR);
        alert.setTitle("Erreur");
        alert.setHeaderText("Erreur de calcul");
        alert.setContentText(message);
        alert.showAndWait();
    }
    
    @FXML
    private void clearHistory() {
        formuleManager.clearHistory();
        historyList.getItems().clear();
        resultLabel.setText("");
    }
}
```

## Vue FXML

### formule_view.fxml
```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<BorderPane xmlns:fx="http://javafx.com/fxml"
            fx:controller="com.inventaire.controller.FormuleController"
            stylesheets="@../css/formule_style.css">
    
    <!-- Titre -->
    <top>
        <VBox styleClass="header">
            <padding>
                <Insets top="20" right="20" bottom="20" left="20"/>
            </padding>
            <Label text="Calculateur de Formules" styleClass="title"/>
            <Label text="Calculs automatiques pour la gestion d'inventaire" 
                   styleClass="subtitle"/>
        </VBox>
    </top>
    
    <!-- Contenu principal -->
    <center>
        <VBox spacing="15" styleClass="content">
            <padding>
                <Insets top="20" right="20" bottom="20" left="20"/>
            </padding>
            
            <!-- Sélection de formule -->
            <VBox spacing="5">
                <Label text="Sélectionner une formule:" styleClass="label"/>
                <ComboBox fx:id="formulaSelector" prefWidth="400"/>
            </VBox>
            
            <!-- Description -->
            <VBox spacing="5">
                <Label text="Description:" styleClass="label"/>
                <TextArea fx:id="descriptionArea" 
                         editable="false" 
                         wrapText="true"
                         prefHeight="60"
                         styleClass="description"/>
            </VBox>
            
            <!-- Paramètres -->
            <VBox spacing="5">
                <Label text="Paramètres:" styleClass="label"/>
                <GridPane fx:id="parametersGrid" 
                         hgap="10" 
                         vgap="10"
                         styleClass="parameters"/>
            </VBox>
            
            <!-- Bouton calcul -->
            <HBox spacing="10" alignment="CENTER">
                <Button fx:id="calculateButton" 
                       text="Calculer" 
                       onAction="#calculateFormula"
                       styleClass="calculate-button"/>
            </HBox>
            
            <!-- Résultat -->
            <VBox spacing="5">
                <Label text="Résultat:" styleClass="label"/>
                <Label fx:id="resultLabel" 
                      styleClass="result"
                      minHeight="40"/>
            </VBox>
        </VBox>
    </center>
    
    <!-- Historique -->
    <right>
        <VBox spacing="10" styleClass="sidebar">
            <padding>
                <Insets top="20" right="20" bottom="20" left="20"/>
            </padding>
            
            <Label text="Historique" styleClass="sidebar-title"/>
            <ListView fx:id="historyList" 
                     prefWidth="300"
                     VBox.vgrow="ALWAYS"/>
            <Button text="Effacer l'historique" 
                   onAction="#clearHistory"
                   styleClass="clear-button"/>
        </VBox>
    </right>
    
</BorderPane>
```

## Exemple d'Utilisation

```java
public class ExampleUsage {
    public static void main(String[] args) {
        // Créer le gestionnaire de formules
        FormuleManager manager = new FormuleManager();
        
        // Exemple 1: Calculer le coût total
        Map<String, Double> params1 = new HashMap<>();
        params1.put("prixAchat", 1500.0);
        params1.put("fraisInstallation", 200.0);
        params1.put("fraisMaintenance", 300.0);
        
        CalculResult result1 = manager.calculate("Coût Total", params1);
        System.out.println(result1); // Affiche: Coût Total: 2000.00 €
        
        // Exemple 2: Calculer l'amortissement
        Map<String, Double> params2 = new HashMap<>();
        params2.put("prixAchat", 2000.0);
        params2.put("dureeVieAnnees", 5.0);
        params2.put("anneesUtilisation", 2.0);
        params2.put("valeurResiduelle", 200.0);
        
        CalculResult result2 = manager.calculate(
            "Amortissement Linéaire", params2
        );
        System.out.println(result2); // Affiche la valeur après 2 ans
        
        // Exemple 3: Calculer la valeur résiduelle
        Map<String, Double> params3 = new HashMap<>();
        params3.put("prixAchat", 1500.0);
        params3.put("tauxDepreciation", 0.15); // 15% par an
        params3.put("anneesUtilisation", 3.0);
        
        CalculResult result3 = manager.calculate(
            "Valeur Résiduelle", params3
        );
        System.out.println(result3);
    }
}
```

## Tests Unitaires

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class FormulaTest {
    
    @Test
    public void testCoutTotal() {
        CoutTotalFormula formula = new CoutTotalFormula();
        Map<String, Double> params = new HashMap<>();
        params.put("prixAchat", 1000.0);
        params.put("fraisInstallation", 100.0);
        params.put("fraisMaintenance", 50.0);
        
        double result = formula.calculate(params);
        assertEquals(1150.0, result, 0.01);
    }
    
    @Test
    public void testAmortissementLineaire() {
        AmortissementLineaireFormula formula = 
            new AmortissementLineaireFormula();
        Map<String, Double> params = new HashMap<>();
        params.put("prixAchat", 1000.0);
        params.put("dureeVieAnnees", 5.0);
        params.put("anneesUtilisation", 2.0);
        params.put("valeurResiduelle", 100.0);
        
        double result = formula.calculate(params);
        assertEquals(640.0, result, 0.01); // 1000 - ((1000-100)/5 * 2)
    }
    
    @Test
    public void testValidationParametres() {
        CoutTotalFormula formula = new CoutTotalFormula();
        Map<String, Double> params = new HashMap<>();
        params.put("prixAchat", -100.0); // Valeur négative
        
        assertFalse(formula.validateParameters(params));
    }
}
```

Ce document fournit une base solide pour implémenter la fonctionnalité formule dans l'application d'inventaire.
