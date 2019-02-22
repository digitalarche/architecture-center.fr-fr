<!-- TEMPLATE FILE - DO NOT ADD METADATA -->

### <a name="governance-of-resources"></a>Gouvernance des ressources

L’application de la gouvernance dans les abonnements provient d’Azure Blueprints et des ressources associées au sein du blueprint.

1. Créez un blueprint Azure nommé « MVP de gouvernance ».
    1. Appliquez l’utilisation des rôles Azure standards.
    2. Permettez aux utilisateurs de s’authentifier uniquement via une implémentation RBAC existante.
    3. Appliquez ce blueprint à tous les abonnements au sein du groupe d’administration.
2. Créez une Azure Policy pour appliquer ou exécuter les éléments suivants  :
    1. Le balisage des ressources requiert des valeurs pour la fonction d’entreprise, la classification des données, le caractère critique, le SLA, l’environnement et l’application.
    2. La valeur de la balise Application doit correspondre au nom du groupe de ressources.
    3. Validez les attributions de rôle pour chaque groupe de ressources et chaque ressource.
3. Publiez le blueprint Azure « MVP de gouvernance » et appliquez-le à chaque groupe d’administration.

Ces modèles permettent aux ressources d’être visibles et suivies, et appliquent la gestion des rôles de base.

### <a name="demilitarized-zone-dmz"></a>Zone démilitarisée (DMZ)

Il est courant que certains abonnements spécifiques demandent un niveau d’accès aux ressources locales. Cela peut être le cas dans les scénarios de migration et de développement, quand certaines ressources dépendantes se trouvent toujours dans le centre de données local. Dans ce cas, le MVP de gouvernance ajoute les meilleures pratiques suivantes :

1. Établissez une zone démilitarisée cloud.
    1. L’[architecture de référence DMZ cloud](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid) établit un modèle de déploiement pour la création d’une passerelle VPN dans Azure.
    2. Vérifiez que la connectivité DMZ et les exigences de sécurité sont en place pour un appareil périphérique dans le centre de données local.
    3. Vérifiez que l’appareil périphérique local est compatible avec les exigences de passerelle VPN Azure.
    4. Une fois la connexion au VPN local vérifiée, capturez le modèle Resource Manager créé par cette architecture de référence.
2. Créez un deuxième blueprint nommé « DMZ ».
    1. Ajoutez le modèle Resource Manager pour la passerelle VPN au blueprint.
3. Appliquez le blueprint DMZ aux abonnements requérant une connectivité locale. Ce blueprint doit être appliqué en plus du blueprint MVP de gouvernance.

Une des principales préoccupations soulevées par les équipes de sécurité informatique et de gouvernance traditionnelle est le risque d’une adoption cloud trop précoce qui mette en péril les ressources existantes. L’approche ci-dessus permet aux équipes d’adoption du cloud de créer et de migrer des solutions hybrides, ce qui réduit les risques pour les ressources locales. Ultérieurement, cette solution temporaire est supprimée.

> [!NOTE]
> La méthode ci-dessus est un point de départ permettant de créer rapidement un MVP de gouvernance de référence. Il ne s’agit que du début du parcours de gouvernance. Des évolutions futures seront nécessaires à mesure que l’entreprise poursuit son adoption du cloud et prend plus de risques dans les domaines suivants :
>
> - Charges de travail critiques
> - Données protégées
> - la gestion des coûts ;
> - Scénarios multi-cloud
>
>Par ailleurs, les détails de ce MVP se basent sur l’exemple de parcours d’une entreprise fictive, qui est décrit dans les articles qui suivent. Nous vous recommandons vivement de consulter les autres articles de cette série avant d’implémenter cette meilleure pratique.
