# Gerg2008-stageSUBLIME
Modélisation du mélange de fluide

Fonction Pression 
AlpharGERG : calcule la fonction de Helmholtz résiduelle, qui encode toutes les interactions non idéales du mélange.
ar[0][1] : c’est la dérivée de la fonction résiduelle par rapport à la densité réduite, qui permet de corriger le facteur de compressibilité par rapport au gaz parfait.
dPdDsave : cette variable (probablement globale ou statique) contient la dérivée de la pression par rapport à la densité, très utile lors des résolutions itératives pour inverser l’équation d’état.


Fonction Densité
Initialisation : estimation initiale de la densité (gaz parfait ou valeur passée).
Boucle de Newton-Raphson sur log(volume) : à chaque itération, on ajuste la densité pour que la pression calculée (via GERG) soit égale à la pression cible.
Relance automatique si la solution diverge ou tombe dans une zone physiquement impossible (2 phases, dérivée négative, etc.).
Tests avancés de stabilité thermodynamique si demandé (iFlag > 0), pour éviter des solutions non physiques.
Sortie : si la convergence échoue, on renvoie la densité gaz parfait, avec un message d’erreur.

Fonction Propriétés 
Calculs préliminaires : masse molaire, Helmholtz idéale (a0) et résiduelle (ar), constantes physiques.
Propriétés de base : pression, facteur Z, dérivées thermodynamiques.
Fonctions d’état : énergie libre de Helmholtz (A), de Gibbs (G), énergie interne (U), enthalpie (H), entropie (S).
Capacités calorifiques : à volume constant (Cv), à pression constante (Cp).
Propriétés avancées : dérivées secondes, coefficient de Joule-Thomson (JT), vitesse du son (W), compressibilité isentropique (Kappa).
Traitement spécial : pour D proche de 0 (limite gaz parfait), certaines propriétés sont ajustées pour éviter des valeurs non physiques.


Fonction Réduction mélange 
Optimisation : Si la composition n’a pas changé, on réutilise les anciens paramètres.
Boucles imbriquées : Les sommes prennent en compte toutes les interactions binaires (voire ternaires dans la version complète GERG).
Facteurs de pondération (xij) : Ils combinent les fractions molaires de chaque paire de composants, avec un facteur F qui gère la symétrie (évite de compter deux fois les mêmes paires).
Paramètres d’interaction (gvij, bvij, gtij, btij) : Matrices contenant les coefficients de réduction pour chaque paire de composants (provenant des données du modèle GERG).
Conversion finale : Le volume de réduction est inversé pour obtenir la densité de réduction.
Stockage des résultats : Les valeurs calculées sont mémorisées pour éviter des calculs inutiles lors des futurs appels avec la même composition.


Fonction partie idéale fonction Helmoltz 
But : calculer la partie idéale de la fonction de Helmholtz (et ses dérivées jusqu'à l'ordre 2), qui correspond à un mélange de gaz parfaits.
Boucles imbriquées :
Sur les composants (i)
Sur les termes liés aux fonctions hyperboliques (j=4 à 7)
Sommes SumHyp0, SumHyp1, SumHyp2 :
Permettent de traiter les contributions des termes hyperboliques dans la formulation analytique (sinh, cosh).
Formules analytiques :
Pour chaque terme, le code distingue si on utilise le sinh (hsn) ou le cosh (hcn), en fonction de l’indice j.
Poids molaire :
Les contributions sont pondérées par les fractions molaires x[i] de chaque composant.
Résultat :
a0[0] = α₀ (fonction de Helmholtz idéale)
a0[1] = dérivée première par rapport à tau (T_r/T)
a0[2] = dérivée seconde



Fonction partie non idéale fonction Helmoltz
Variables réduites : delta (densité réduite), tau (température réduite), calculées pour la composition courante.
Pureté et interactions : On somme d’abord les contributions de chaque fluide pur, puis celles des couples (i,j) du mélange.
Puissances et exponentielles : Calcul anticipé pour l’efficacité lors des sommes.
Tableaux d’indices et coefficients : Tous les indices et coefficients (doik, taup, kpol, etc.) sont issus des paramètres du modèle GERG-2008 pour chaque composant ou chaque couple d’interaction.
Dérivées : Le tableau ar reçoit les valeurs de la fonction d’Helmholtz résiduelle et ses dérivées jusqu’à l’ordre 3 en delta et tau, nécessaires pour le calcul des propriétés thermodynamiques avancées (pression, Cp, Cv, etc.).



Fonction 
Objectif : Préparer les puissances de tau (tau = Tr/T), pondérées par des coefficients, nécessaires pour l'évaluation efficace de la fonction de Helmholtz résiduelle et de ses dérivées dans GERG-2008.
taup[i][k] : Tableau contenant pour chaque composant pur : coefficient × (tau puissance exposant), pour chaque terme polynômial ou exponentiel.
taupijk[mn][k] : Idem pour chaque interaction binaire (mn = numéro de la paire (i,j)).
Optimisation : Pour la plupart des composants (hors exceptions), on réutilise les puissances taup0[k] précalculées afin de gagner du temps.
Cas particuliers : Certains composants (indices 1 à 4, 15, 18, 20) nécessitent un calcul spécifique à chaque appel pour respecter la formulation précise GERG.
Utilité : Ces tableaux sont ensuite utilisés dans AlpharGERG pour former rapidement les sommes de chaque terme sans recalculer les exponentielles à chaque fois.



Fonction pseudo critique
Les tableaux Tc[i] (température critique du composant i) et Dc[i] (densité critique du composant i) doivent être définis ailleurs dans le programme.
NcGERG : nombre de composants considérés dans le modèle GERG.
epsilon : petite valeur pour éviter la division par zéro.
