# IMPORTER LA BASE DE DONNEE : https://www.kaggle.com/datasets/zzettrkalpakbal/full-filled-brain-stroke-dataset

# Nettoyage

### Désactivation du safe mode
SET SQL_SAFE_UPDATES = 0;

### Après avoir explorer les données, nous remarquons que parmis les valeurs dans la colonne d'âge, il y a des décimales. On vérifie donc si il y en a d'autres.
SELECT age from stroke_data
WHERE age LIKE '%.%';

### Suppression des lignes avec des âges décimaux
DELETE FROM stroke_data
WHERE age LIKE '%.%';

### Suppression des lignes avec des valeurs aberrantes pour le niveau de glucose et l'IMC/BMI. En sachant que les résultats retrouvés sont la moyenne du taux de glucose : 127,87 et son écart-type : 59,43. La moyenne de l'IMC qui est de 29,67 et son écart type 2,82.

SELECT 
    AVG(avg_glucose_level) AS avg_glucose,
    STDDEV(avg_glucose_level) AS stddev_glucose,
    AVG(bmi) AS avg_bmi,
    STDDEV(bmi) AS stddev_bmi
FROM stroke_data;

DELETE FROM stroke_data
WHERE 
    (avg_glucose_level < (127.87 - 2 * 59.43) OR avg_glucose_level > (127.87 + 2 * 59.43))
    OR 
    (bmi < (29.67 - 2 * 2.82) OR bmi > (29.67 + 2 * 2.82));

# Analyse statistique

## 1. Statistiques globales sur l'âge

SELECT ROUND(AVG(age),2) AS Age_Moy, MIN(age) AS Age_Min, MAX(age) AS Age_Max
FROM stroke_data;

### Interpretation
### L'âge moyen étant quand même élevée = 53.01, on peut interpreter que les risques d'AVC ou ceux ayant déjà eu des AVC chez un patient est plus fréquent chez une personne âgée ce qui est cohérent par rapport avec les connaissances médicales,
### Etant donnée que l'âge minimum est assez bas, cela peut suggerer que même les enfants peuvent être a risque.
### Un âge maximum élevé est attendu et montre que les mesures préventives doivent se concentrer sur les personnes âgées pour réduire le risque d’AVC

## 2. Statistiques sur le niveau de glucose et l'IMC/BMI

SELECT 
    AVG(avg_glucose_level) AS avg_glucose,
    STDDEV(avg_glucose_level) AS stddev_glucose,
    AVG(bmi) AS avg_bmi,
    STDDEV(bmi) AS stddev_bmi
FROM stroke_data;

### Interpretation
### A REMPLIR

## 4. Moyenne du niveau de glucose par catégorie d’hypertension

WITH GlucoseParHT AS (
    SELECT 
        hypertension, 
        round(AVG(avg_glucose_level),2) AS avg_glucose
    FROM stroke_data
    GROUP BY hypertension)
SELECT 
    CASE 
        WHEN hypertension = 0 THEN 'Pas hypertension' 
        WHEN hypertension = 1 THEN 'Hypertension' 
        ELSE 'Inconnue' 
    END AS hypertension_status, avg_glucose
FROM GlucoseParHT;

### Interprétation
### La légère différence de moyenne des niveaux de glucose entre les patients hypertendus et non hypertendus montre que, dans cette base de données, l’hypertension n’a pas d’impact majeur sur les niveaux de glucose. 


## 5. Fréquence des AVC par statut tabagique

SELECT 
    smoking_status,
    COUNT(*) AS total_patients,
    SUM(stroke) AS stroke_cases,
    AVG(stroke) * 100 AS avg_stroke
FROM stroke_data
GROUP BY smoking_status;

### Nous pouvons remarquer que pour les patient n'ayant jamais fumé présentent un taux d'AVC modéré (15.0%) qui suggère que le fait de ne jamais avoir fumé est associé à un risque relativement plus bas d'AVC.
### Pour les anciens fumeurs, avec un taux d'AVC de 28.26%, les anciens fumeurs présentent un risque plus élevé que les non-fumeurs mais moins que les patients avec un statut inconnu (28,26%)
### Etonnamment, les fumeurs ont le taux le plus bas d'AVC avec 5,8% environ. Il est possible que d'autres facteurs, comme des comportements de santé ou des traitements médicaux, influencent ces résultats.

## 6. Fréquence des AVC par tranche d'âge

SELECT 
    CASE 
        WHEN age < 30 THEN 'Moins de 30'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
        WHEN age BETWEEN 40 AND 49 THEN '40-49'
        WHEN age BETWEEN 50 AND 59 THEN '50-59'
        WHEN age BETWEEN 60 AND 69 THEN '60-69'
        WHEN age >= 70 THEN '70 et plus'
    END AS groupe_age,
    COUNT(*) AS total_patients,
    SUM(stroke) AS stroke_cases,
    AVG(stroke) * 100 AS stroke_rate
FROM stroke_data
GROUP BY groupe_age;

### Interpretation


## 7. Analyse par niveau de glucose

### Nous cherchons à établir des tranches d'âges dans notre cas de figure pour déterminer la tranche d'âge ayant le plus de personnes exposées au risque de maladie cérébrale.

SELECT 
    CASE 
        WHEN age BETWEEN 0 AND 14 THEN 'enfant (0-14 ans)'
        WHEN age BETWEEN 15 AND 24 THEN 'jeune (15-24 ans)'
        WHEN age BETWEEN 25 AND 64 THEN 'adulte (25-64 ans)'
        WHEN age >= 65 THEN 'doyen (65 ans et plus)'
        ELSE 'Non défini'
    END AS tranche_age,
    count(*) AS total,
    round(avg(avg_glucose_level), 2) as avg_glucose,
    round(avg(bmi),2) as avg_bmi
FROM projet.projetSQL
GROUP BY tranche_age ;

### Interprétation

## Ces données montrent les statistiques agrégées des individus répartis par tranche d'âge, avec le nombre total d'individus, la moyenne de leur niveau de glucose (`avg_glucose`) et la moyenne de leur indice de masse corporelle (`avg_bmi`). Voici une interprétation des résultats :


## 1. Adultes (25-64 ans) :
## Interprétation : Cette tranche d'âge montre un niveau de glucose relativement élevé (130.91) et un BMI moyen légèrement au-dessus de la limite de surpoids (un BMI de 31.01 est dans la catégorie de l'obésité selon l'OMS). Cela suggère que les adultes de ce groupe peuvent être à risque de problèmes de santé liés à l'obésité et au diabète, étant donné les niveaux élevés de glucose et de BMI.

## 2. Doyens (65 ans et plus) :
## Interprétation : Les doyens montrent un niveau de glucose assez proche de celui des adultes, mais légèrement plus faible. Leur BMI moyen (29.53) est juste en dessous de 30, donc à la limite de l'obésité. Le fait que les doyens aient un niveau de glucose et un BMI élevés pourrait indiquer un risque de maladies chroniques telles que le diabète de type 2 ou des maladies cardiovasculaires.

## 3. Enfants (0-14 ans) :
## Interprétation : Le niveau de glucose des enfants est plus bas que celui des adultes et des doyens (122.31), ce qui est typique puisque les enfants sont généralement en meilleure santé métabolique. Leur BMI moyen est également dans une fourchette normale (21.08), ce qui suggère qu'ils ont une santé relativement bonne par rapport aux autres tranches d'âge.

## 4. Jeunes (15-24 ans) :
## Interprétation : Les jeunes ont le niveau de glucose moyen le plus bas (98.21), ce qui est un bon indicateur de leur santé métabolique. Leur BMI moyen est de 26.7, ce qui les place dans la catégorie de surpoids selon les normes de l'OMS. Il pourrait y avoir des signes de surpoids chez certains jeunes, mais cela n'a pas encore affecté leur niveau de glucose de manière significative.

## Interprétation globale :
## Les adultes (25-64 ans) et les doyens (65 ans et plus) ont des niveaux de glucose et de BMI plus élevés, ce qui peut être un signe de risque accru de maladies métaboliques (comme le diabète de type 2 ou l'obésité).
## Les enfants (0-14 ans) ont des indicateurs de santé plus équilibrés, avec un niveau de glucose et un BMI normaux.
## Les jeunes (15-24 ans) ont un glucose bas mais un BMI un peu plus élevé, ce qui pourrait indiquer une tendance vers un surpoids, mais ils ne montrent pas encore de signes de détérioration métabolique.
## Ces données mettent en évidence des différences dans les risques de santé potentiels en fonction de l'âge, avec des populations plus âgées présentant un risque plus élevé de problèmes de santé liés à des niveaux élevés de glucose et de BMI.

## 8. Analyse par IMC

SELECT AVG(bmi) AS average_bmi
FROM health_data;

SELECT 
    COUNT(CASE WHEN stroke = 1 THEN 1 END) AS stroke_count,
    COUNT(*) AS total_count,
    (COUNT(CASE WHEN stroke = 1 THEN 1 END) * 100.0 / COUNT(*)) AS stroke_percentage
FROM projet.projetSQL
WHERE bmi >= 30 ;

### Interprétation :

## stroke_count (15) : Cela indique qu'il y a eu **15 cas d'AVC** (accidents vasculaires cérébraux) parmi les personnes évaluées. Ce chiffre représente le nombre total de personnes dans cette catégorie d'IMC qui ont subi un AVC.

## total_count (94) : Ce chiffre représente le **total de 94 personnes** dans la catégorie d'IMC considérée. Cela inclut toutes les personnes, qu'elles aient ou non des antécédents d'AVC.

## stroke_percentage (15.95745) : Ce pourcentage signifie qu'environ **15.96 %** des personnes dans cette catégorie d'IMC ont subi un AVC. C'est un indicateur clé pour évaluer le risque d'AVC dans cette population.

### Analyse IMC :

## Contexte de l'IMC : Pour analyser les résultats dans le contexte de l'IMC, il serait pertinent de comparer ce pourcentage avec celui des autres catégories d'IMC (anorexie, sous-poids, poids normal, surpoids, obésité).
  
## Évaluation du risque : Si ce pourcentage est significativement plus élevé que celui des autres catégories, cela peut suggérer un lien entre cette catégorie d'IMC et un risque accru d'AVC.
  
## Conclusion : Une prévalence d’AVC de presque 16 % dans cette catégorie d'IMC pourrait indiquer la nécessité d'interventions ciblées pour réduire ce risque. Cela pourrait inclure des programmes de sensibilisation à la santé, des conseils nutritionnels ou d'autres mesures de prévention.

SELECT 
    CASE
        WHEN bmi <= 17.5 THEN 'Anorexie' 
        WHEN bmi BETWEEN 17.5 AND 18.5 THEN 'Sous-poids'
        WHEN bmi BETWEEN 18.5 AND 24.9 THEN 'Poids normal'
        WHEN bmi BETWEEN 25 AND 29.9 THEN 'Sur-poids'
        ELSE 'Obésité'
    END AS bmi_category,
    COUNT(*) AS count,
	COUNT(*) * 100.0 / (SELECT COUNT(*) FROM projet.projetSQL) AS percentage
FROM projet.projetSQL 
GROUP BY bmi_category ;

### Interprétation :

## Obésité (count : 103, percentage : 52.82 %) :
## Interprétation : Plus de la moitié (52.82 %) des individus dans cette analyse sont classés comme obèses. Cela suggère que l'obésité est un problème de santé significatif dans la population étudiée, et cela peut être lié à divers risques pour la santé, y compris des maladies cardiovasculaires, le diabète, et, comme mentionné précédemment, un risque accru d'AVC.

## Sur-poids (count : 83, percentage : 42.56 %) :
## Interprétation : Près de 43 % des individus sont classés comme en surpoids. Cela, combiné à la proportion élevée d'obésité, indique une tendance préoccupante vers un surpoids généralisé dans cette population. Ce groupe pourrait également être à risque accru de comorbidités associées à l'obésité.

## Poids normal (count : 9, percentage : 4.62 %) :
## Interprétation : Seulement 4.62 % des individus sont dans la catégorie de poids normal. Cela souligne le fait que la majorité de la population étudiée souffre d'une surcharge pondérale (surpoids ou obésité), ce qui pose des questions sur la santé publique et les mesures de prévention à mettre en place.

### Analyse IMC :

## Contexte global :
##  - Ces résultats montrent une prévalence élevée d'obésité et de surpoids dans la population, indiquant un besoin urgent d'interventions de santé publique pour aborder ces problèmes. 
## - Une faible proportion de personnes avec un poids normal peut indiquer des habitudes alimentaires malsaines, un manque d'activité physique, ou d'autres facteurs contribuant à l'augmentation du poids.

## Implications pour la santé :
##  - Les taux élevés d'obésité et de surpoids sont souvent associés à des risques accrus de maladies chroniques, ce qui souligne l'importance d'évaluer les habitudes de vie et les facteurs environnementaux.

## Conclusion :
## Les résultats révèlent une situation préoccupante en matière de santé publique, nécessitant des actions visant à promouvoir un mode de vie sain, notamment des programmes de sensibilisation à la nutrition et à l'exercice physique. Si tu souhaites approfondir certains aspects ou explorer d'autres analyses, fais-le moi savoir !

SELECT 
    CASE 
        WHEN bmi < 18.5 THEN 'Underweight'
        WHEN bmi BETWEEN 18.5 AND 24.9 THEN 'Normal weight'
        WHEN bmi BETWEEN 25 AND 29.9 THEN 'Overweight'
        ELSE 'Obesity'
    END AS bmi_category,
   round(AVG(hypertension),2) AS avg_hypertension,
   round(AVG(heart_disease),2) AS avg_heart_disease,
    COUNT(*) AS count
FROM projet.projetSQL
GROUP BY bmi_category;


### Interprétation :
## Obésité (avg_hypertension : 0.38, avg_heart_disease : 0.18, count : 103) :
## - Hypertension : Une moyenne de 0.38 indique que 38 % des personnes obèses ont des antécédents d'hypertension. Cela suggère un lien fort entre l'obésité et l'hypertension, ce qui est attendu étant donné que l'obésité est un facteur de risque majeur pour l'hypertension.
## - Maladies cardiaques : Avec une moyenne de 0.18, cela signifie que 18 % des personnes obèses ont des antécédents de maladies cardiaques. Cela indique également un risque accru de problèmes cardiaques dans ce groupe.

## 2. Surpoids (avg_hypertension : 0.10, avg_heart_disease : 0.17, count : 83) :
## - Hypertension : Une moyenne de 0.10 indique que 10 % des personnes en surpoids ont des antécédents d'hypertension, ce qui est significativement plus bas que chez les obèses, mais indique toujours un risque relatif.
## - Maladies cardiaques : Une moyenne de 0.17 signifie que 17 % des personnes en surpoids ont des antécédents de maladies cardiaques, ce qui est proche de la proportion observée chez les obèses.

## 3. Poids normal (avg_hypertension : 0.00, avg_heart_disease : 0.00, count : 9) :
## - Hypertension et maladies cardiaques : Aucune personne dans cette catégorie n'a d'antécédents d'hypertension ou de maladies cardiaques. Cela suggère que les personnes avec un poids normal ont un risque nettement plus faible de développer ces conditions.

### Analyse IMC :
## - Contexte global :
## - Les résultats montrent une tendance claire : plus l'IMC augmente, plus les risques d'hypertension et de maladies cardiaques augmentent.
## - L'absence d'antécédents dans la catégorie "poids normal" souligne l'importance d'un poids sain pour la prévention des maladies chroniques.

## - Implications pour la santé :
## - Les résultats soulignent la nécessité de cibler les populations obèses et en surpoids pour des interventions de santé préventives. Cela peut inclure des programmes de gestion du poids, de nutrition, et d'exercice physique.

### Conclusion :
## Les résultats mettent en évidence un lien entre l'IMC, l'hypertension et les maladies cardiaques. Ces observations soulignent l'importance de la gestion du poids dans la prévention de maladies graves. Si tu souhaites explorer d'autres aspects ou analyses, n'hésite pas à demander !



