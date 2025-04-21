# POC avec Donnees dynamiques avec l’outil BIRT en Developpement java (Spring Boot)

## 1 Mise en Oeuvre de l'outil avec Spring Boot 
[Lien vers la procedure de Mise en oeuvre de BIRT avec Spring Boot](https://github.com/ludovicfranck/Procedure-de-mise-en-oeuvre-de-l-outil-BIRT-avec-Spring-Boot.git)

## 2 Modèle de données
```java
package com.example.birtdemo.model;

public class Employee {
    private Long id;
    private String name;
    private String department;
    private Double salary;

    // Constructeurs, getters et setters
    public Employee(Long id, String name, String department, Double salary) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
    }

    // Getters et setters...
}

```
## 3 Servive étendu pour les données Java 

```java
public byte[] generateEmployeeReport(String format, List<Employee> employees, 
                                   HttpServletRequest request, 
                                   HttpServletResponse response) 
    throws BirtException, IOException {
    
    String reportPath = servletContext.getRealPath("/reports/employee_report.rptdesign");
    IReportRunnable design = birtEngine.openReportDesign(reportPath);
    IRunAndRenderTask task = birtEngine.createRunAndRenderTask(design);
    
    // Configuration du rendu
    RenderOption options = getRenderOptions(format);
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    options.setOutputStream(outputStream);
    task.setRenderOption(options);
    
    // Passage des données Java
    task.getAppContext().put("employeeData", employees);
    task.getAppContext().put(EngineConstants.APPCONTEXT_CLASSLOADER_KEY, 
                           BirtReportService.class.getClassLoader());
    
    // Paramètres
    HashMap<String, Object> params = new HashMap<>();
    params.put("reportTitle", "Employee Report");
    task.setParameterValues(params);
    
    task.run();
    task.close();
    
    return outputStream.toByteArray();
}

private RenderOption getRenderOptions(String format) {
    switch (format.toLowerCase()) {
        case "pdf":
            return new PDFRenderOption();
        case "html":
            HTMLRenderOption htmlOptions = new HTMLRenderOption();
            htmlOptions.setHtmlRtLFlag(false);
            return htmlOptions;
        case "xls":
            return new EXCELRenderOption();
        default:
            throw new IllegalArgumentException("Format non supporté: " + format);
    }
}
```

## 4 Endpoint pour le rapport employé
```java
@GetMapping("/employees/{format}")
public ResponseEntity<byte[]> generateEmployeeReport(
        @PathVariable String format,
        HttpServletRequest request,
        HttpServletResponse response) {
    
    List<Employee> employees = List.of(
        new Employee(1L, "John Doe", "IT", 5000.0),
        new Employee(2L, "Jane Smith", "HR", 4500.0),
        new Employee(3L, "Robert Johnson", "Finance", 6000.0)
    );
    
    try {
        byte[] reportContent = birtReportService
            .generateEmployeeReport(format, employees, request, response);
        
        return ResponseEntity.ok()
                .contentType(getMediaType(format))
                .header("Content-Disposition", "attachment; filename=employees." + format)
                .body(reportContent);
    } catch (Exception e) {
        return ResponseEntity.internalServerError().build();
    }
}
```

## 5 Tester l'application
- Lancer l'application Spring Boot
- Acceder aux Endpoints :
  - [](http://localhost:8080/api/reports/simple_report/pdf) pour le rapport simple en pdf
  - [](http://localhost:8080/api/reports/simple_report/html) pour le rapport simple en html
  - [](http://localhost:8080/api/reports/employees/pdf) pour le rapport employé en pdf

## 6. Bonnes pratiques et optimisation

Cache des rapports : Cachez les designs de rapport compilés pour améliorer les performances

Gestion des ressources : Assurez-vous de toujours fermer les tâches et le moteur BIRT

 Configuration : Configurez le chemin des logs BIRT pour le dépannage

 Sécurité : Validez les paramètres d'entrée pour éviter les attaques par injection

 Concurrence : Le moteur BIRT n'est pas thread-safe, utilisez un pool d'engines pour les hautes performances

 ## 7. Alternatives pour la production

 Pour des déploiements en production:
 - Utilisez Pentaho BA Server pour les rapports complexes
 - Considérez une génération asynchrone des rapports
 - Mettez en place un système de files d'attente pour les gros rapports

Cette implémentation montre comment intégrer efficacement Pentaho Reporting dans une application Spring Boot, avec prise en charge de plusieurs formats de sortie et injection de données dynamiques.
New chat
