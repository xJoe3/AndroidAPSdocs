# Fonctions OpenAPS

## Autosens

* Autosens est un algorithme qui examine les écarts de glycémie (positives/négatives/neutres).
* Il va essayer de déterminer à quel point vous êtes sensible/résistant en fonction de ces écarts.
* L'implémentation oref dans **OpenAPS** utilise une combinaison de 24 et 8 heures de données. Il utilise celui qui le est plus sensible.
* AndroidAPS n'exécute que la version 8 heures (pour activer les RNS) ou 24 heures en tant qu'option utilisateur.
* Le changement de canule ou le changement de profil réinitialisera le ratio Autosens à 0%.
* Autosens ajuste votre basal et votre SI pour vous (c.-à-d. qu'il imite ce que fait un changement de profil).
* Si vous mangez continuellement des glucides sur une période prolongée, l'Autosens sera moins efficace pendant cette période car les glucides sont exclus les calculs des écarts de glycémie.

## Super Micro Bolus (SMB)

SMB, la version courte de 'Super Micro Bolus', est la dernière fonctionnalité de OpenAPS (depuis 2018) inclue dans l'algorithme Oref1. Contrairement à OpenAPS AMA, le SMB n'utilise pas les débits de base temporaires pour contrôler la glycémie, mais surtout les **microbolus de toute petite taille**. dans les cas où AMA ajouterait 1.0 UI d'insuline à l'aide d'un débit de base temporaire, SMB délivre plusieurs Super Micro Bolus en petites étapes à **5 minutes d'intervalle**, par ex. 0.4 UI, 0.3 UI, 0.2 UI and 0.1 UI. Dans le même temps (pour des raisons de sécurité) le véritable taux basal est mis à 0 UI/h pour une certaine durée afin d'éviter un surdosage (**'zéro-temp'**). Cela permet au système d'ajuster la glycémie plus rapidement qu'avec l'augmentation du débit de base temporaire de l'AMA.

Grâce aux SMB, il peut être suffisant pour un repas faible en glucides d'informer le système de la quantité de glucides prévue et de laisser faire le reste par AAPS. Cependant, cela peut conduire à des pics postprandiaux plus élevés car le pré-bolus n’est pas possible. Ou vous pouvez donner, si vous avez besoin d'un pré-bolus, un **bolus de départ bolus**, qui couvre **seulement une partie** des glucides (par ex. 2/3 de la quantité estimée) et vous laissez les SMB couvrir le reste.

La fonctionnalité SMB contient des mécanismes de sécurité:

1. La plus grande dose de SMB ne peut être que la plus petite valeur entre :
    
    * la valeur correspondant au débit de base actuel (ajusté par autotune / autosens) pour la durée définie dans "Max minutes de base pour limiter le SMB", par ex. la quantité de basale pour les 30 prochaines minutes, ou
    * la moitié de la quantité d'insuline actuellement requise, ou
    * la partie restante de votre maxIA renseignée dans les paramètres.

2. Vous remarquerez probablement souvent de faibles débits de base temporaires (appelées 'faibles temp') ou des DBT à 0 U/h (appélés 'zéro-temp'). C'est par conception pour des raisons de sécurité et cela n'a aucun effets négatif si le profil est défini correctement. La courbe d'IA est plus significative que les débits de base temporaires.

3. Des calculs supplémentaires sont effectués pour prédire l'évolution de la glycémie, par ex. RNS (ou Repas Non Signalés). Même si aucun glucide n'est renseigné par l'utilisateur, RNS peut détecter automatiquement une augmentation significative des niveaux de glycémie liés à des repas, l'adrénaline ou d'autres facteurs et essaiera de les ajuster avec des SMB. Pour être en sécurité, cela marche aussi dans l'autre sens et peut arrêter les SMB plus tôt si une chute rapide inattendue de la glycémie survient. C'est pourquoi RNS doit toujours être activé avec les SMB.

**Vous devez avoir terminé l'[Objectif 10](../Usage/Objectives#objective-10-enabling-additional-oref1-features-for-daytime-use-such-as-super-micro-bolus-smb) pour utiliser les SMB.**

Voir aussi : [Documentation OpenAPS pour oref1 SMB](https://openaps.readthedocs.io/en/latest/docs/Customize-Iterate/oref1.html) et [Tim's Info sur les SMB](http://www.diabettech.com/artificial-pancreas/understanding-smb-and-oref1/).

### Max. U/h pour le débit temp Basal (OpenAPS "max-basal")

Ce paramètre de sécurité détermine le débit de base temporaire maximal que la pompe à insuline peut délivrer. La valeur doit être la même dans la pompe et dans les AAPS et doit être au moins égale à 3 fois le débit de base le plus élevé.

Exemple :

Le débit de base le plus élevé de votre profil au cours de la journée est de 1,00 U/h. Alors la valeur de max-basal recommandée est d'au moins 3 U/h.

Mais vous ne pouvez pas choisir n'importe quelle valeur. AAPS limite la valeur en 'dur' en fonction de l'âge du patient que vous avez sélectionné dans les paramètres. La valeur permise est la plus faible pour les enfants et la plus élevée pour les adultes résistants à l’insuline.

AndroidAPS limite la valeur ainsi :

* Enfant : 2
* Adolescent : 5
* Adulte : 10
* Adulte résistant à l'insuline : 12

### Maximum Insuline Active IA pour OpenAPS \[U\] (OpenAPS "maxIA")

Cette valeur détermine quelle valeur de maxIA doit être prise en compte par AAPS en mode boucle fermée. Si l'IA en cours (par exemple après un bolus de repas) est supérieure à la valeur définie, la boucle arrêtera d'administrer de l'insuline jusqu'à ce que la l'IA soit inférieure à la valeur limite renseignée.

En utilisant OpenAPS SMB, maxIA est calculé différemment de OpenAPS AMA. Dans AMA, maxIA était juste un paramètre de sécurité pour l'IA de la basal, alors qu'en mode SMB, il inclut également l'IA des bolus. Un bon départ est

    maxIA = moyenne bolus repas + 3 x max basal quotidien
    

Soyez prudent et patient et modifiez les paramètres petit à petit. C'est différent pour tout le monde et dépend aussi de la Dose Totale d'Insuline (DTI) moyenne quotidienne. Pour des raisons de sécurité, il y a une limite, qui dépend de l'âge du patient. La limite "en dur" de maxIA est plus élevée que celle de l'AMA.

* Enfant : 3
* Adolescent : 7
* Adulte : 12
* Adulte résistant à l'insuline : 25

Voir aussi la [documentation OpenAPS pour SMB](https://openaps.readthedocs.io/en/latest/docs/Customize-Iterate/oref1.html#understanding-smb).

### Activer AMA Autosens

Ici, vous pouvez choisir si vous voulez utiliser la [détection de sensibilité](../Configuration/Sensitivity-detection-and-COB.md) 'autosens' ou non.

### Activer SMB

Ici, vous pouvez activer ou désactiver complètement la fonction SMB.

### Activer SMB avec les glucides

SMB ne fonctionne que lorsqu'il y a des glucides actifs (GA).

### Activer SMB avec les cibles temporaires

SMB fonctionne quand il y a une cible temporaire faible ou élevée (Repas imminent, Activité, Hypo, Personnalisé)

### Activer SMB avec cibles temp. hautes

SMB is working when there is a high temporary target active (activity, hypo). This option can limit other SMB Settings, i.e. if ‘SMB with temp targets’ is enabled and ‘SMB with high temp targets’ is deactivated, SMB just works with low and not with high temp targets. It is the same for enabled SMB with COB: if 'SMB with high temp target' is deactivated, there is no SMB with high temp target even if COB is active.

### Activer en permanence les SMB

SMB is working always (independent of COB, temp targets or boluses). For safety reasons, this option is just possibly for BG sources with a nice filtering system for noisy data. For now, it just works with a Dexcom G5, if using the Dexcom App (patched) or “native mode” in xDrip+. If a BG value has a too large deviation, the G5 doesn’t send it and waits for the next value in 5 minutes.

For other CGM/FGM like Freestyle Libre, ‘SMB always’ is deactivated until xDrip+ has a better noise smoothing plugin. You can find more [here](../Usage/Smoothing-Blood-Glucose-Data-in-xDrip.md).

### Activer SMB après ingestion de glucides

SMB is working for 6h after carbohydrates , even if COB is at 0. For safety reasons, this option is just possibly for BG sources with a nice filtering system for noisy data. For now, it just works with a Dexcom G5, if using the Dexcom App (patched) or “native mode” in xDrip+. If a BG value has a too large deviation, the G5 doesn’t send it and waits for the next value in 5 minutes.

For other CGM/FGM like Freestyle Libre, 'SMB always' is deactivated until xDrip+ has a better noise smoothing plugin. You can find [more information here](../Usage/Smoothing-Blood-Glucose-Data-in-xDrip.md).

### Max. minutes de basal pour limiter le SMB

This is an important safety setting. This value determines how much SMB can be given based on the amount of basal insulin in a given time, when it is not covered by COBs.

This makes the SMB more aggressive. For the beginning, you should start with the default value of 30 minutes. After some experience, you can increase the value with 15 minutes steps and watch how these changes are affecting.

It is recommended not to set the value higher than 90 minutes, as this would lead to a point where the algorithm might not be able to adjust a decreasing BG with 0 IE/h basal ('zero-temp'). You should also set alarms, especially if you are still testing new settings, which warns you before running into hypos.

Default value: 30 min.

### Activer RNS

With this option enabled, the SMB algorithm can recognize unannounced meals. This is helpful, if you forget to tell AndroidAPS about your carbs or estimate your carbs wrong and the amount of entered carbs is wrong or if a meal with lots of fat and protein has a longer duration than expected. Without any carb entry, UAM can recognize fast glucose increasments caused by carbs, adrenaline, etc, and tries to adjust it with SMBs. This also works the opposite way: if there is a fast glucose decreasement, it can stop SMBs earlier.

**Par conséquent, les RNS doivent toujours être activés lors de l'utilisation de SMB.**

### High temp-target raises sensitivity

If you have this option enabled, the insulin sensitivity will be increased while having a temporary target over 100 mg/dl or 5.6 mmol/l. This means, the ISF will rise while IC and basal will decrease.

### Low temp-target lowers sensitivity

If you have this option enabled, the insulin sensitivity will be decreased while having a temporary target lower than 100 mg/dl or 5.6 mmol/l. This means, the ISF will decrease while IC and basal will rise.

### Paramètres Avancés

**Utiliser delta basé sur moyenne courte** Si vous activez cette fonction, AndroidAPS utilise une moyenne courte des variations de glycémie sur les 15 dernières minutes, ce qui correspond généralement à la moyenne des trois dernières valeurs. Cela aide AndroidAPS à travailler plus régulièrement avec des sources de données bruyantes comme xDrip+ et Libre.

**Max daily safety multiplier** This is an important safety limit. The default setting (which is unlikely to need adjusting) is 3. This means that AndroidAPS will never be allowed to set a temporary basal rate that is more than 3x the highest hourly basal rate programmed in a user’s pump. Example: if your highest basal rate is 1.0 U/h and max daily safety multiplier is 3, then AndroidAPS can set a maximum temporary basal rate of 3.0 U/h (= 3 x 1.0 U/h).

Default value: 3 (shouldn’t be changed unless you really need to and know, what you are doing)

**Current Basal safety multiplier** This is another important safety limit. The default setting (which is also unlikely to need adjusting) is 4. This means that AndroidAPS will never be allowed to set a temporary basal rate that is more than 4x the current hourly basal rate programmed in a user’s pump.

Default value: 4 (shouldn’t be changed unless you really need to and know, what you are doing)

* * *

## Assistance Améliorée Repas (AAR)

AAR, la version abrégée de "Assistance Améliorée Repas" est une fonctionnalité OpenAPS de 2017 (oref0). OpenAPS Advanced Meal Assist (AMA) allows the system to high-temp more quickly after a meal bolus if you enter carbs reliably.

**You will need to have completed [objective 9](../Usage/Objectives#objective-9-enabling-additional-oref0-features-for-daytime-use-such-as-advanced-meal-assist-ama) to use this feature**

You can find more information in the [OpenAPS documentation](http://openaps.readthedocs.io/en/latest/docs/walkthrough/phase-4/advanced-features.html#advanced-meal-assist-or-ama).

### Max U/hr a Temp Basal can be set to (OpenAPS "max-basal")

This safety setting helps AndroidAPS from ever being capable of giving a dangerously high basal rate and limits the temp basal rate to x U/h. Il est conseillé de definir cette valuer de facon raisonnable et sensée. A good recommendation is to take the highest basal rate in your profile and multiply it by 4 and at least 3. For example, if the highest basal rate in your profile is 1.0 U/h you could multiply that by 4 to get a value of 4 U/h and set the 4 as your safety parameter.

You cannot chose any value: For safety reason, there is a 'hard limit', which depends on the patient age. The 'hard limit' for maxIOB is lower in AMA than in SMB. For children, the value is the lowest while for insulin resistant adults, it is the biggest.

The hardcoded parameters in AndroidAPS are:

* Enfant : 2
* Adolescent : 5
* Adulte : 10
* Adulte résistant à l'insuline : 12

### Maximum basal IOB OpenAPS can deliver \[U\] (OpenAPS "max-iob")

This parameter limits the maximum of basal IOB where AndroidAPS still works. If the IOB is higher, it stops giving additional basal insulin until the basal IOB is under the limit.

The default value is 2, but you should be rise this parameter slowly to see how much it affects you and which value fits best. C'est différent pour tout le monde et dépend aussi de la Dose Totale d'Insuline (DTI) moyenne quotidienne. Pour des raisons de sécurité, il y a une limite, qui dépend de l'âge du patient. The 'hard limit' for maxIOB is lower in AMA than in SMB.

* Enfant : 3
* Adolescent : 5
* Adulte : 7
* Adulte résistant à l'insuline : 12

### Activer AMA Autosens

Here, you can chose, if you want to use the [sensitivity detection](../Configuration/Sensitivity-detection-and-COB.md) autosense or not.

### Autosens ajuste aussi les cibles temp

Si cette option est activée, autosens peut également ajuster les cibles (à côté du débit de base, SI et G/I). Cela permet à AndroidAPS d'être plus ou moins "agressif". La cible réelle peut être atteinte plus rapidement avec ceci.

### Paramètres Avancés

**Utiliser delta basé sur moyenne courte** Si vous activez cette fonction, AndroidAPS utilise une moyenne courte des variations de glycémie sur les 15 dernières minutes, ce qui correspond généralement à la moyenne des trois dernières valeurs. Cela aide AndroidAPS à travailler plus régulièrement avec des sources de données bruyantes comme xDrip+ et Libre.

**Max daily safety multiplier** This is an important safety limit. The default setting (which is unlikely to need adjusting) is 3. This means that AndroidAPS will never be allowed to set a temporary basal rate that is more than 3x the highest hourly basal rate programmed in a user’s pump, or, if enabled, determined by autotune. Example: if your highest basal rate is 1.0 U/h and max daily safety multiplier is 3, then AndroidAPS can set a maximum temporary basal rate of 3.0 U/h (= 3 x 1.0 U/h).

Default value: 3 (shouldn’t be changed unless you really need to and know, what you are doing)

**Current Basal safety multiplier** This is another important safety limit. The default setting (which is also unlikely to need adjusting) is 4. This means that AndroidAPS will never be allowed to set a temporary basal rate that is more than 4x the current hourly basal rate programmed in a user’s pump, or, if enabled, determined by autotune.

Default value: 4 (shouldn’t be changed unless you really need to and know, what you are doing)

**Bolus snooze dia divisor** The feature “bolus snooze” works after a meal bolus. AAPS doesn’t set low temporary basal rates after a meal in the period of the DIA divided by the “bolus snooze”-parameter. The default value is 2. That means with a DIA of 5h, the “bolus snooze” would be 5h : 2 = 2.5h long.

Default value: 2

* * *

## Meal Assist (MA)

### Max U/hr a Temp Basal can be set to (OpenAPS "max-basal")

This safety setting helps AndroidAPS from ever being capable of giving a dangerously high basal rate and limits the temp basal rate to x U/h. Il est conseillé de definir cette valuer de facon raisonnable et sensée. A good recommendation is to take the highest basal rate in your profile and multiply it by 4 and at least 3. For example, if the highest basal rate in your profile is 1.0 U/h you could multiply that by 4 to get a value of 4 U/h and set the 4 as your safety parameter.

You cannot chose any value: For safety reason, there is a 'hard limit', which depends on the patient age. The 'hard limit' for maxIOB is lower in MA than in SMB. For children, the value is the lowest while for insulin resistant adults, it is the biggest.

The hardcoded parameters in AndroidAPS are:

* Enfant : 2
* Adolescent : 5
* Adulte : 10
* Adulte résistant à l'insuline : 12

### Maximum basal IOB OpenAPS can deliver \[U\] (OpenAPS "max-iob")

This parameter limits the maximum of basal IOB where AndroidAPS still works. If the IOB is higher, it stops giving additional basal insulin until the basal IOB is under the limit.

The default value is 2, but you should be rise this parameter slowly to see how much it affects you and which value fits best. C'est différent pour tout le monde et dépend aussi de la Dose Totale d'Insuline (DTI) moyenne quotidienne. Pour des raisons de sécurité, il y a une limite, qui dépend de l'âge du patient. The 'hard limit' for maxIOB is lower in MA than in SMB.

* Enfant : 3
* Adolescent : 5
* Adulte : 7
* Adulte résistant à l'insuline : 12

### Paramètres Avancés

**Utiliser delta basé sur moyenne courte** Si vous activez cette fonction, AndroidAPS utilise une moyenne courte des variations de glycémie sur les 15 dernières minutes, ce qui correspond généralement à la moyenne des trois dernières valeurs. Cela aide AndroidAPS à travailler plus régulièrement avec des sources de données bruyantes comme xDrip+ et Libre.

**Bolus snooze dia divisor** The feature “bolus snooze” works after a meal bolus. AAPS doesn’t set low temporary basal rates after a meal in the period of the DIA divided by the “bolus snooze”-parameter. The default value is 2.That means with a DIA of 5h, the “bolus snooze” would be 5h : 2 = 2.5h long.

Default value: 2