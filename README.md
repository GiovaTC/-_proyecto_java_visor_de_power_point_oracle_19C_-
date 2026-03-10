# -_proyecto_-generador_de_variables_aleatorias_en_java_- :.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9ffba1af-e68d-4da6-a8fb-276c3b7ee5cd" />  

## 📊 Proyecto Java:
Generador de Variables Aleatorias con GUI + Excel 

Proyecto académico que implementa un generador de variables aleatorias utilizando diferentes distribuciones de probabilidad, con una interfaz gráfica en Swing y exportación de resultados a Excel (.xlsx).

✔ Interfaz gráfica con Swing
✔ Generación de variables aleatorias
✔ Distribuciones:

- Uniforme
- Normal (Gaussiana)
- Binomial
- Poisson

✔ Tabla de resultados
✔ Exportación de resultados a Excel (.xlsx)

Este tipo de proyecto es ampliamente utilizado en simulación estadística y probabilidad aplicada.

🧰 Tecnologias Utilizadas:

- Java 17 / Java 21

- Swing (Interfaz gráfica)

- Apache POI (Exportación a Excel)

- IntelliJ IDEA

```
📁 Estructura del Proyecto
src
 ├── model
 │     RandomResult.java
 │
 ├── service
 │     RandomDistributionService.java
 │
 ├── util
 │     ExcelExporter.java
 │
 └── ui
       RandomGeneratorGUI.java
```

1️⃣ Modelo:
RandomResult.java

Clase modelo que representa el resultado de una variable aleatoria generada.
```
package model;

public class RandomResult {

    private String distribution;
    private double value;

    public RandomResult(String distribution, double value) {
        this.distribution = distribution;
        this.value = value;
    }

    public String getDistribution() {
        return distribution;
    }

    public double getValue() {
        return value;
    }
```

2️⃣ Servicio de Distribuciones:
RandomDistributionService.java

Clase encargada de generar números aleatorios según diferentes distribuciones de probabilidad.
```
package service;

import java.util.Random;

public class RandomDistributionService {

    private Random random = new Random();

    public double uniform(double min, double max) {
        return min + (max - min) * random.nextDouble();
    }

    public double normal(double mean, double std) {
        return mean + std * random.nextGaussian();
    }

    public int binomial(int trials, double p) {

        int success = 0;

        for (int i = 0; i < trials; i++) {
            if (random.nextDouble() < p) {
                success++;
            }
        }

        return success;
    }

    public int poisson(double lambda) {

        double L = Math.exp(-lambda);
        int k = 0;
        double p = 1;

        do {
            k++;
            p *= random.nextDouble();
        } while (p > L);

        return k - 1;
    }
}
```

3️⃣ Exportar Resultados a Excel:

Para exportar los resultados se utiliza la librería Apache POI.
```
Dependencia Maven
<dependency>
 <groupId>org.apache.poi</groupId>
 <artifactId>poi-ooxml</artifactId>
 <version>5.2.5</version>
</dependency>
```
ExcelExporter.java

Clase encargada de exportar los resultados generados a un archivo Excel (.xlsx).
```
package util;

import model.RandomResult;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileOutputStream;
import java.util.List;

public class ExcelExporter {

    public static void export(List<RandomResult> results) {

        try {

            Workbook workbook = new XSSFWorkbook();
            Sheet sheet = workbook.createSheet("Resultados");

            Row header = sheet.createRow(0);
            header.createCell(0).setCellValue("Distribucion");
            header.createCell(1).setCellValue("Valor");

            int rowIndex = 1;

            for (RandomResult r : results) {

                Row row = sheet.createRow(rowIndex++);

                row.createCell(0).setCellValue(r.getDistribution());
                row.createCell(1).setCellValue(r.getValue());
            }

            FileOutputStream fileOut = new FileOutputStream("resultados.xlsx");
            workbook.write(fileOut);

            fileOut.close();
            workbook.close();

            System.out.println("Excel generado correctamente");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

4️⃣ Interfaz Gráfica Swing:
RandomGeneratorGUI.java

Implementa la interfaz gráfica que permite seleccionar la distribución, ingresar parámetros y visualizar los resultados.
```
package ui;

import model.RandomResult;
import service.RandomDistributionService;
import util.ExcelExporter;

import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class RandomGeneratorGUI extends JFrame {

    private JComboBox<String> distributionBox;
    private JTextField param1;
    private JTextField param2;

    private JTable table;
    private DefaultTableModel model;

    private RandomDistributionService service = new RandomDistributionService();
    private List<RandomResult> results = new ArrayList<>();

    public RandomGeneratorGUI() {

        setTitle("Generador de Variables Aleatorias");
        setSize(600,400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        JPanel topPanel = new JPanel();

        distributionBox = new JComboBox<>(new String[]{
                "Uniforme","Normal","Binomial","Poisson"
        });

        param1 = new JTextField(5);
        param2 = new JTextField(5);

        JButton generateButton = new JButton("Generar");
        JButton exportButton = new JButton("Exportar Excel");

        topPanel.add(new JLabel("Distribucion"));
        topPanel.add(distributionBox);
        topPanel.add(new JLabel("Parametro 1"));
        topPanel.add(param1);
        topPanel.add(new JLabel("Parametro 2"));
        topPanel.add(param2);
        topPanel.add(generateButton);
        topPanel.add(exportButton);

        add(topPanel, BorderLayout.NORTH);

        model = new DefaultTableModel(new String[]{"Distribucion","Valor"},0);
        table = new JTable(model);

        add(new JScrollPane(table), BorderLayout.CENTER);

        generateButton.addActionListener(e -> generate());
        exportButton.addActionListener(e -> export());

    }

    private void generate() {

        String dist = (String) distributionBox.getSelectedItem();

        double p1 = Double.parseDouble(param1.getText());
        double value = 0;

        if(dist.equals("Uniforme")) {

            double p2 = Double.parseDouble(param2.getText());
            value = service.uniform(p1,p2);

        } else if(dist.equals("Normal")) {

            double p2 = Double.parseDouble(param2.getText());
            value = service.normal(p1,p2);

        } else if(dist.equals("Binomial")) {

            double p2 = Double.parseDouble(param2.getText());
            value = service.binomial((int)p1,p2);

        } else if(dist.equals("Poisson")) {

            value = service.poisson(p1);
        }

        RandomResult result = new RandomResult(dist,value);

        results.add(result);

        model.addRow(new Object[]{dist,value});
    }

    private void export() {

        ExcelExporter.export(results);

        JOptionPane.showMessageDialog(this,"Excel exportado");
    }

    public static void main(String[] args) {

        SwingUtilities.invokeLater(() -> {
            new RandomGeneratorGUI().setVisible(true);
        });
    }
}
```

🖥 Resultado Visual Esperado:

- La interfaz tendrá:

- Selector de distribución

- Campos para parámetros

- Botón Generar

- Tabla de resultados

- Botón Exportar Excel
  
```
+--------------------------------------------------+
| Distribucion  [Normal ▼]  Param1  Param2  Generar |
|--------------------------------------------------|
| Distribucion | Valor                              |
|--------------------------------------------------|
| Normal       | 3.45                               |
| Poisson      | 5                                  |
| Uniforme     | 7.82                               |
+--------------------------------------------------+
                 [Exportar Excel]
```
                 
📈 Aplicaciones de este Programa:

Este generador permite simular distintos fenómenos probabilísticos:

⏳ Tiempo de espera

Distribución Poisson

⚙️ Errores en sistemas

Distribución Binomial

⚡ Eventos raros

Distribución Poisson

🌎 Fenómenos naturales

Distribución Normal / . 
