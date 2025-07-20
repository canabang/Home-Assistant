<img width="1859" height="642" alt="image" src="https://github.com/user-attachments/assets/b20800b0-70e5-4d58-840e-16b924152fc6" />
# 🐾 Pet Feeder - Home Assistant Integration

Une gestion complète pour gérer automatiquement un distributeur de nourriture pour animaux via Home Assistant et Zigbee2MQTT.

Sachant que le poids de la portion et le poids total du réservoir peuvent varier suivant les croquettes utilisées. 
Tout va se passer dans HA, plus besoin de jongler entre HA pour les informations et Z2M pour les réglages.

## ✨ Fonctionnalités principales

### 🎯 Gestion automatique des portions
- **Calcul intelligent** : Les portions sont automatiquement calculées en fonction du poids unitaire et de l'objectif quotidien
- **Répartition optimisée** : Distribution sur 3 repas (Matin 28%, Midi 32%, Soir 40%)
- **Recalcul automatique** : Mise à jour immédiate lors du changement des paramètres

### 📊 Suivi de la consommation
- **Monitoring en temps réel** : Suivi du dernier repas distribué et du total quotidien
- **Moyennes calculées** : Consommation moyenne basée sur l'historique réel
- **Compteurs avancés** : Total cumulé et consommation quotidienne

### 📦 Gestion intelligente du stock
- **Estimation automatique** : Calcul du stock restant basé sur la consommation
- **Alertes visuelles** : Indicateurs de niveau bas avec code couleur
- **Prédiction** : Estimation des jours restants avant rupture de stock
- **Détection de rechargement** : Reset automatique lors du remplissage

### 🎛️ Interface utilisateur complète
- **Dashboard dédié** : Interface claire avec jauges et graphiques
- **Contrôles manuels** : Distribution test, reset compteurs, recalcul forcé
- **Statistiques avancées** : Analyse détaillée de la consommation

## 🏗️ Architecture

### Entités principales

#### Numbers (du distributeur)
- `number.pet_feeder_portion_weight` : **Poids unitaire par portion (g)** - Paramètre clé pour tous les calculs

#### Input Numbers
- `pet_feeder_target_daily_weight` : Objectif quotidien (20-150g)
- `pet_feeder_stock_estime` : Stock estimé actuel
- `pet_feeder_max_capacity` : Capacité maximale du réservoir (variable selon croquettes)
- `pet_feeder_daily_consumption_counter` : Compteur consommation quotidienne
- `pet_feeder_total_distributed` : Total distribué (cumulatif)

#### Sensors calculés
- `portions_calculees_*` : Portions pour chaque repas (matin/midi/soir)
- `pourcentage_croquette_restant` : Niveau du réservoir en %
- `consommation_quotidienne_moyenne` : Moyenne de consommation
- `jours_restants_nourriture` : Prédiction avant rupture

#### Input Boolean
- `pet_feeder_stock_was_low` : Indicateur technique de stock bas

### Automatisations intelligentes

1. **Recalcul automatique des portions**
   - Déclencheur : Changement du poids unitaire (`number.pet_feeder_portion_weight`) ou objectif quotidien
   - Action : Recalcul des portions et mise à jour MQTT du planning de distribution

2. **Mise à jour du stock**
   - Déclencheur : Nouveau repas distribué
   - Actions : gère la baisse du stock, mise à jour des compteurs

3. **Détection de rechargement**
   - Condition : Passage de <300g à >950g
   - Action : Reset des compteurs et indicateurs

4. **Surveillance stock bas**
   - Seuil : 300g
   - Action : Activation de l'indicateur d'alerte

## 🚀 Installation

### Prérequis
- Home Assistant avec Zigbee2MQTT fonctionnel
- Distributeur Pet Feeder compatible Zigbee déjà intégré
- Cartes personnalisées : `mini-graph-card`, `multiple-entity-row`, `bar-card`

### Configuration

1. **Copier/Créer les fichiers de configuration** (ou ajouter le contenu à vos fichier existant) :
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
   - Initialiser le stock estimé après un remplissage complet

## 📱 Utilisation

### Configuration des repas
1. **Calibrez le poids unitaire** via `number.pet_feeder_portion_weight` (varie selon les croquettes)
2. Définissez l'**objectif quotidien** selon les besoins de votre animal
3. Le système calcule automatiquement la répartition optimale
4. Les portions sont envoyées automatiquement au distributeur

### Surveillance du stock
- Consultez le **niveau du réservoir** via la jauge visuelle
- Surveillez les **jours restants** pour anticiper le rechargement
- Les alertes visuelles vous préviennent quand le stock est bas

### Suivi de la consommation
- Visualisez la **consommation en temps réel** vs l'objectif
- Analysez les **tendances** via les moyennes calculées
- Consultez l'**historique cumulatif** des distributions

## ⚙️ Paramètres configurables

| Paramètre | Description | Valeur par défaut |
|-----------|-------------|------------------|
| **Poids unitaire** | Poids par portion (g) - **Variable selon croquettes** | À calibrer |
| Objectif quotidien | Quantité cible par jour | 85g |
| Capacité réservoir | Volume maximum - **Dépend des croquettes** | Variable |
| Seuil stock bas | Niveau d'alerte | 300g |
| Répartition matin | % du total quotidien | 28% |
| Répartition midi | % du total quotidien | 32% |
| Répartition soir | % du total quotidien | 40% |

## 🔧 Calibration selon les croquettes

### Poids unitaire par portion
Le poids d'une portion varie significativement selon :
- **Taille des croquettes** : Plus elles sont grosses, plus une portion pèse lourd
- **Densité** : Croquettes light vs standard
- **Form factor** : Forme ronde, triangulaire, etc.

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

### Alertes intelligentes
- **Stock bas** : Notification visuelle et technique
- **Rechargement détecté** : Reset automatique des compteurs
- **Écart objectif** : Comparaison consommation réelle vs cible

### Contrôles manuels directement depuis HA
- **Distribution test** : Déclenchement manuel d'un repas
- **Reset compteurs** : Remise à zéro des statistiques quotidiennes
- **Recalcul forcé** : Mise à jour manuelle des portions
- **Ajustement poids unitaire** : Modification directe sans passer par Z2M

### Statistiques détaillées
- Consommation moyenne sur période
- Prédiction de durée avant rupture
- Suivi cumulatif des distributions
- Analyse des écarts par rapport à l'objectif

## 🎨 Interface visuelle

L'interface comprend :
- **Jauges en temps réel** : Niveau stock et jours restants
- **Cartes de suivi** : Consommation quotidienne et programmation
- **Indicateurs colorés** : Alertes visuelles selon les seuils
- **Contrôles interactifs** : Actions manuelles directement depuis l'interface HA

## 🔧 Personnalisation

Le système est entièrement modulaire et peut être adapté :
- Modification des horaires de distribution
- Ajustement des pourcentages de répartition
- Personnalisation des seuils d'alerte
- Adaptation aux différents types de croquettes
- Extension avec d'autres capteurs (balance, caméra, etc.)

## 📝 Logs et debug

Le système génère des logs détaillés pour :
- Chaque recalcul de portions
- Mise à jour des stocks
- Distribution des repas
- Détection des rechargements

Consultez les logs système Home Assistant pour le debug.

## 💡 Avantages de cette intégration

✅ **Centralisation complète** : Plus besoin de basculer entre HA et Z2M  
✅ **Adaptation automatique** : S'ajuste selon le type de croquettes  
✅ **Précision optimale** : Calibration fine du poids unitaire  
✅ **Surveillance intelligente** : Prédictions et alertes avancées  
✅ **Interface unifiée** : Contrôle total depuis le dashboard HA  

---

*Ce projet permet une gestion complètement automatisée de l'alimentation de vos animaux avec un suivi précis et des alertes intelligentes, tout en s'adaptant parfaitement aux différents types de croquettes. 🐱🐶*
