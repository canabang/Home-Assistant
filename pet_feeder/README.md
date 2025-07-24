<img width="1853" height="820" alt="image" src="https://github.com/user-attachments/assets/2554303f-317c-4197-90c9-a2878f52693f" />

# 🐾 Pet Feeder - Home Assistant Integration

Une gestion complète pour gérer automatiquement un distributeur de nourriture pour animaux via Home Assistant et Zigbee2MQTT.

Sachant que le poids de la portion et le poids total du réservoir peuvent varier suivant les croquettes utilisées. 
Tout va se passer dans HA, plus besoin de jongler entre HA pour les informations et Z2M pour les réglages.

## ✨ Fonctionnalités principales

### 🎯 Gestion automatique des portions
- **Calcul intelligent** : Les portions sont automatiquement calculées en fonction du poids unitaire et de l'objectif quotidien
- **Répartition optimisée** : Distribution sur 3 repas (Matin 28%, Midi 32%, Soir 40%)
- **Recalcul automatique** : Mise à jour immédiate lors du changement des paramètres
- **Application automatique** : Les nouvelles portions sont directement envoyées au distributeur via MQTT

### 📊 Suivi de la consommation avancé
- **Double méthode de calcul** : Estimation classique ET calcul précis basé sur données réelles
- **Monitoring en temps réel** : Suivi du dernier repas distribué et du total quotidien
- **Moyennes calculées** : Consommation moyenne basée sur l'historique réel depuis le refill
- **Compteurs multiples** : Quotidien, cumulé depuis refill, et total général
- **Comparaison intelligente** : Interface permettant de comparer les estimations théoriques vs réelles

### 📦 Gestion intelligente du stock
- **Estimation automatique** : Calcul du stock restant basé sur la consommation
- **Double prédiction** : Jours restants calculés selon méthode classique ET précise
- **Alertes visuelles** : Indicateurs de niveau bas avec code couleur
- **Détection de rechargement** : Reset automatique lors du remplissage (seuil <300g → >950g)
- **Initialisation manuelle** : Possibilité de déclencher manuellement un nouveau cycle de suivi
- **Horodatage précis** : Suivi des dates et durées depuis le dernier remplissage

### 🎛️ Interface utilisateur complète
- **Dashboard dédié** : Interface claire avec jauges et graphiques
- **Contrôles manuels** : Distribution test, reset compteurs, recalcul forcé
- **Statistiques avancées** : Analyse détaillée avec comparaison des méthodes
- **Indicateurs de précision** : Niveau de fiabilité selon la durée des données collectées

## 🏗️ Architecture

### Entités principales

#### Numbers (du distributeur)
- `number.pet_feeder_portion_weight` : **Poids unitaire par portion (g)** - Paramètre clé pour tous les calculs

#### Input Numbers
- `pet_feeder_target_daily_weight` : Objectif quotidien (20-150g, défaut: 60g)
- `pet_feeder_stock_estime` : Stock estimé actuel
- `pet_feeder_max_capacity` : Capacité maximale du réservoir (variable selon croquettes, défaut: 1500g)
- `pet_feeder_daily_consumption_counter` : Compteur consommation quotidienne
- `pet_feeder_cumulative_distributed_since_refill` : Poids cumulé depuis dernier remplissage
- `pet_feeder_total_distributed` : Total distribué (cumulatif général)
- `pet_feeder_refill_date` : Timestamp du dernier remplissage

#### Sensors calculés
- `portions_calculees_*` : Portions pour chaque repas (matin/midi/soir)
- `total_portions_calculees` : Total des portions programmées avec détails
- `pourcentage_croquette_restant` : Niveau du réservoir en %
- `consommation_quotidienne_moyenne` : Consommation du jour en cours
- `consommation_moyenne_reelle` : **Consommation réelle basée sur données cumulatives**
- `jours_restants_nourriture` : Prédiction classique avant rupture
- `jours_restants_nourriture_methode_cumulative` : **Prédiction précise basée sur consommation réelle**
- `poids_reel_distribue_aujourd_hui` : Dernier repas distribué en grammes

#### Input Boolean
- `pet_feeder_stock_was_low` : Indicateur technique de stock bas
- `pet_feeder_manual_refill_trigger` : Déclencheur pour initialisation manuelle d'un nouveau cycle

### Automatisations intelligentes

1. **Recalcul automatique des portions**
   - Déclencheur : Changement du poids unitaire (`number.pet_feeder_portion_weight`) ou objectif quotidien (`input_number.pet_feeder_target_daily_weight`)
   - Action : Recalcul des portions et mise à jour MQTT automatique du planning de distribution

2. **Mise à jour compteurs unifiée**
   - Déclencheur : Nouveau repas distribué (variation de `sensor.pet_feeder_weight_per_day`)
   - Actions : Gestion simultanée de tous les compteurs (quotidien, cumulé, stock, total général)

3. **Détection de rechargement automatique**
   - Condition : Passage de <300g à >950g
   - Action : Reset automatique des compteurs et indicateurs, initialisation nouveau cycle

4. **Initialisation manuelle remplissage**
   - Déclencheur : Activation de `input_boolean.pet_feeder_manual_refill_trigger`
   - Action : Initialisation manuelle d'un nouveau cycle de suivi

5. **Reset quotidien automatique**
   - Déclencheur : Minuit chaque jour
   - Action : Remise à zéro du compteur quotidien

6. **Surveillance stock bas**
   - Seuil : 300g
   - Action : Activation de l'indicateur d'alerte

## 🚀 Installation

### Prérequis
- Home Assistant avec Zigbee2MQTT fonctionnel
- Distributeur Pet Feeder compatible Zigbee déjà intégré
- Cartes personnalisées : `mini-graph-card`, `multiple-entity-row`, `bar-card`, `template-entity-row`

### Configuration

1. **Copier/Créer les fichiers de configuration** (ou ajouter le contenu à vos fichiers existants) :
   ```yaml
   # Dans configuration.yaml
   input_number: !include input_number.yaml
   input_boolean: !include input_boolean.yaml
   template: !include template.yaml
   automation: !include automations.yaml
   ```

2. **Ajouter le dashboard** :
   - Copier le contenu de `pet_feeder/dashboard.yaml` dans un nouveau dashboard

3. **Adapter la configuration MQTT** :
   - Vérifier le topic MQTT : `zigbee2mqtt02/Pet feeder/set`
   - Ajuster selon votre configuration Zigbee2MQTT

4. **Configuration initiale** :
   - **Calibrer le poids unitaire** via `number.pet_feeder_portion_weight` (crucial pour la précision)
   - Définir la capacité maximale du réservoir selon le type de croquettes
   - Configurer l'objectif quotidien de votre animal

5. **Initialisation du système** :
   - Effectuer un premier remplissage complet du réservoir
   - Définir le stock estimé à la capacité maximale
   - Activer `input_boolean.pet_feeder_manual_refill_trigger` pour initialiser le premier cycle de suivi
   - Vérifier que `pet_feeder_refill_date` est correctement défini

## 📱 Utilisation

### Configuration des repas
1. **Calibrez le poids unitaire** via `number.pet_feeder_portion_weight` (varie selon les croquettes)
2. Définissez l'**objectif quotidien** selon les besoins de votre animal
3. Le système calcule automatiquement la répartition optimale
4. Les portions sont envoyées automatiquement au distributeur via MQTT

### Surveillance du stock avancée
- Consultez le **niveau du réservoir** via la jauge visuelle
- Comparez les **deux méthodes d'estimation** (classique vs précise)
- Surveillez les **jours restants** pour anticiper le rechargement
- Les alertes visuelles vous préviennent quand le stock est bas
- Suivez la **précision des estimations** selon la durée des données

### Suivi de la consommation intelligent
- Visualisez la **consommation en temps réel** vs l'objectif
- Analysez les **tendances** via les moyennes réelles calculées
- Consultez l'**historique cumulatif** depuis le dernier refill
- Comparez **consommation théorique vs réelle**

### Double système de calcul
- **Estimation classique** : Basée sur l'objectif quotidien théorique
- **Calcul précis** : Basé sur la consommation réelle mesurée depuis le refill  
- **Basculement intelligent** : Le système privilégie le calcul précis quand suffisamment de données sont disponibles
- **Indicateurs de fiabilité** : Niveau de précision affiché selon la durée des données collectées

## ⚙️ Paramètres configurables

| Paramètre | Description | Valeur par défaut |
|-----------|-------------|------------------|
| **Poids unitaire** | Poids par portion (g) - **Variable selon croquettes** | À calibrer |
| Objectif quotidien | Quantité cible par jour | 60g |
| Capacité réservoir | Volume maximum - **Dépend des croquettes** | 1500g |
| Seuil stock bas | Niveau d'alerte | 300g |
| Seuil rechargement | Détection remplissage automatique | 950g |
| Répartition matin | % du total quotidien | 28% |
| Répartition midi | % du total quotidien | 32% |
| Répartition soir | % du total quotidien | 40% |

## 🔧 Calibration selon les croquettes

### Poids unitaire par portion
Le poids d'une portion varie significativement selon :
- **Taille des croquettes** : Plus elles sont grosses, plus une portion pèse lourd
- **Densité** : Croquettes light vs standard
- **Forme factor** : Forme ronde, triangulaire, etc.

**Méthode de calibration recommandée :**
1. Déclencher manuellement 10 portions
2. Peser le total distribué
3. Diviser par 10 pour obtenir le poids moyen
4. Ajuster `number.pet_feeder_portion_weight` avec cette valeur

### Capacité du réservoir
La capacité en poids varie aussi selon les croquettes :
- **Croquettes légères** : ~800g max
- **Croquettes standard** : ~1000g max  
- **Croquettes denses** : ~1500g max

Ajustez `pet_feeder_max_capacity` après un remplissage complet.

## 🛠️ Fonctionnalités avancées

### Système de calcul dual
- **Estimation classique** : Utilisée en début de cycle ou en cas de données insuffisantes
- **Calcul précis** : Privilégié dès que suffisamment de données réelles sont disponibles
- **Comparaison temps réel** : Interface permettant de voir les deux estimations côte à côte
- **Auto-basculement** : Le système choisit automatiquement la méthode la plus fiable

### Alertes intelligentes
- **Stock bas** : Notification visuelle et technique avec seuils configurables
- **Rechargement détecté** : Reset automatique des compteurs avec logging détaillé
- **Écart objectif** : Comparaison consommation réelle vs cible avec pourcentages
- **Précision données** : Indicateurs de fiabilité selon la durée du cycle

### Contrôles manuels avancés depuis HA
- **Distribution test** : Déclenchement manuel d'un repas
- **Reset compteurs** : Remise à zéro sélective des statistiques
- **Initialisation manuelle** : Déclenchement d'un nouveau cycle de suivi
- **Recalcul forcé** : Mise à jour manuelle des portions
- **Ajustement direct** : Modification des paramètres sans passer par Z2M

### Statistiques détaillées
- **Consommation moyenne sur période** : Calculs basés sur données réelles
- **Prédiction double** : Durée avant rupture selon les deux méthodes
- **Suivi cumulatif** : Historique depuis le dernier remplissage
- **Analyse des écarts** : Comparaison objectif vs réalité avec tendances
- **Horodatage précis** : Suivi temporel des cycles et événements

## 🎨 Interface visuelle avancée

L'interface comprend :
- **Jauges en temps réel** : Niveau stock et jours restants avec codes couleur
- **Cartes de suivi dual** : Consommation quotidienne ET analyse sur plusieurs jours
- **Comparaison des méthodes** : Affichage côte à côte des estimations classique vs précise
- **Indicateurs colorés** : Alertes visuelles selon les seuils avec dégradés
- **Contrôles interactifs** : Actions manuelles directement depuis l'interface HA
- **Indicateurs de précision** : Niveau de fiabilité selon la durée des données collectées
- **Graphiques historiques** : Tendances et évolutions avec `mini-graph-card`

## 🔧 Personnalisation

Le système est entièrement modulaire et peut être adapté :
- **Modification des horaires** : Ajustement des heures de distribution
- **Pourcentages personnalisés** : Répartition des repas selon vos besoins
- **Seuils configurables** : Adaptation des alertes et détections
- **Multi-croquettes** : Gestion de différents types avec profils
- **Extensions possibles** : Intégration avec d'autres capteurs (balance, caméra, etc.)
- **Méthodes de calcul** : Ajustement des algorithmes selon vos préférences

## 📝 Logs et debug

Le système génère des logs détaillés pour :
- **Chaque recalcul de portions** : Avec détail des valeurs et calculs
- **Mise à jour des stocks** : Suivi de tous les compteurs simultanément
- **Distribution des repas** : Tracking complet avec poids réels
- **Détection des rechargements** : Automatiques et manuels avec contexte
- **Comparaison des méthodes** : Performance et écarts entre estimations
- **Alertes de cohérence** : Vérification de la logique des données

Consultez les logs système Home Assistant (`Developer Tools > Logs`) pour le debug détaillé.

## 💡 Avantages de cette intégration

✅ **Centralisation complète** : Plus besoin de basculer entre HA et Z2M  
✅ **Adaptation automatique** : S'ajuste selon le type de croquettes avec précision  
✅ **Double intelligence** : Calculs théoriques ET basés sur données réelles  
✅ **Précision optimale** : Calibration fine avec suivi cumulatif avancé  
✅ **Surveillance prédictive** : Estimations multiples avec indicateurs de fiabilité  
✅ **Interface unifiée** : Contrôle total depuis le dashboard HA avec comparaisons  
✅ **Auto-apprentissage** : Le système devient plus précis avec le temps  
✅ **Robustesse** : Gestion des cas d'erreur et fallback automatique  

## 🔍 Différences avec les solutions basiques

**Avantages par rapport aux intégrations simples :**
- Suivi cumulatif précis depuis chaque refill
- Double méthode de calcul avec basculement intelligent
- Gestion automatique des cycles de remplissage
- Interface comparative des estimations
- Indicateurs de précision et de fiabilité
- Logs détaillés pour troubleshooting avancé
- Adaptation dynamique selon les croquettes utilisées

---

*Ce projet permet une gestion complètement automatisée de l'alimentation de vos animaux avec un suivi précis, des prédictions intelligentes et des alertes avancées, tout en s'adaptant parfaitement aux différents types de croquettes. Le système évolue et devient plus précis avec le temps grâce au double calcul théorique/réel. 🐱🐶*
---

*Ce projet permet une gestion complètement automatisée de l'alimentation de vos animaux avec un suivi précis et des alertes intelligentes, tout en s'adaptant parfaitement aux différents types de croquettes. 🐱🐶*
