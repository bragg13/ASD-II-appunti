# === Programmazione Dinamica ===
## DOMINO - numero di possibili disposizioni di qualcosa
```py
int domino(int n){
    DP = new int[0...n]
    DP[0] = 1
    DP[1] = 1
    
    for i=2 to n:
        DP[i] = DP[i-1] + DP[i-2]
    return DP[n]
}
```

## HATEVILLE - prendo/non prendo: quantita massima che posso raccogliere
> posso prendere la donazione di i (skippando precedente e successiva) o non prenderla (passando alla prossima)
```py
int hateville(int D[], int n){
    DP = new int[0...n]
    DP[0] = 0
    DP[1] = D[1]

    for i=2 to n:
        DP[i] = max(DP[i-1], DP[i-2]+D[i]) # non prendo - prendo
    return DP[n]
}
```

## KNAPSACK - massimizzare il profitto ottenuto, con constraint del peso massimo
> DP[i][c] è il massimo profitto ottenibile dai primi i oggetti per capacità massima c => DP[n][C]
```py
int knapsack(int peso[], int valore[], int n, int C){
    DP = new int[0...n][0...C]
    for i=0 to n:
        DP[i][0] = 0                # elemento zero
    
    for c=0 to C:
        DP[0][c] = 0                # zaino con capacità 0
    
    for i=1 to n:
        for c=1 to C:
            if peso[i] <= c:        # lo zaino contiene l'oggetto
                DP[i][c] = max(DP[i-1][c-peso[i]] + valore[i], DP[i-1][c])  # prendo l'oggetto: segno il valore, diminuisco la capacita, passo al prossimo
            else:
                DP[i][c] = DP[i-1][c]                                       # non prendo l'oggetto: passo al prossimo
    return DP[n][C]
}
```


## KNAPSACK senza limiti - posso selezionare piu volte lo stesso elemento
> utilizo memoization e memorizzo l'indice da cui deriva il massimo per ricostruire la soluzione
```py
int knapsack(int peso[], int valore[], int n, int C){
    DP = new int[0...C]
    pos = new int[0...C]
    for i=0 to C:       # inizializzo tutto a -1
        DP[i] = -1
        pos[i] = -1
    knapsackRec(peso, valore, n, C, DP, pos)
    return solution(peso, C, pos)
}
```

> definisco DP[c] il massimo profitto che posso ottenere dagli n oggetti in uno zaino di capacità c
```py
int knapsackRec(int peso[], int valore[], int n, int c, int DP[], int pos[]){
    if c==0:
        return 0        # sono arrivato a capacità 0, non ci sta nulla
    
    if DP[c] == -1:     # non ancora modificato
        DP[c] = 0       # alla peggio ho profitto 0
        for i=1 to n:
            if peso[i] <= c:
                int val = knapsackRec(peso, valore, n, c-peso[i], DP, pos) + valore[i]
                if val >= DP[c]:
                    DP[c] = val
                    pos[c] = i

    return DP[c]
}
```


## SOTTOSEQUENZA COMUNE MASSIMALE - date due sequenze T ed U, quanto sono simili tra loro?
```py
int lcs(int T[], int U[], int n, int m){
    DP = new int[0...n][0...m]
    for i=0 to n:
        DP[i][0] = 0
    for j=0 to m:
        DP[0][j] = 0

    for i=1 to n:
        for j=1 to m:
            if T[i] == S[j]:
                DP[i][j] = DP[i-1][j-1] + 1   # se gli ultimi caratteri sono uguali, calcolo la scm snza contarli e sommo 1 (length)
            else:
                DP[i][j] = max( DP[i-1][j], DP[i][j-1] ) # se son diversi, calcolo la migliore che avrei togliendo l'ultimo carattere ad ognuno
    return DP[n][m]
}
```


## STRING MATCHING APPROSSIMATO - dato un pattern p e un testo t, esiste un'occorrenza k-approssimata di p in t [con k minimo]?
> approssimato significa che ci son due caratteri diversi, che un carattere di p non è in t o viceversa [per un max di k errori]
> DP[i][j] è il minimo valore k per cui esiste un'occorrenza k-approssimata di p(i) in t(j), che termina in j
```py
int strMatch(int P[], int T[], int m, int n){
    DP = new int [0...m][0...n]

    for j=0 to n:
        DP[0][j] = 0        # caso base i=0
    for i=1 to m:           
        DP[i][0] = i        # caso base j=0
    
    for i=1 to m:
        for j=1 to n:
            DP[i][j] = min( DP[i-1][j-1] + (P[i] == T[j]) ? 0 : 1,      # 0 se son uguali, altrimenti aggiungo 1 all'approssimazione
                            DP[i-1]][j] + 1,                            # pattern diminuito
                            DP[i][j-1] + 1 )                            # testo diminuito
    int pos = 0
    for j=1 to n:
        if DP[m][j] < DP[m][pos]:                   # il risultato è il minimo dell'ultima riga
            pos = j
    return pos
}
```


## PRODOTTO DI CATENA DI MATRICI - trovare una parentesizzazione ottima per minimizzare il costo del prodotto
> DP[i][j] è il minimo numero di moltiplicazioni scalari da fare per calcolare A[i...j]. 
> se i==j, DP[i][j]=0, altrimenti il numero di molt è dato DP[i][k]+DP[k+1][j] + il costo per moltiplicare. Ne cerco il minimo;
> posso trovare un k provandoli tutti; non posso riempire una riga dopo l'altra
> DP[i][j] tiene il numero di molt necessarie per moltiplicare A[i...j]
> LAST[i][j] tiene il valore k dell'ultimo prodotto tale che minimizzi il costo del sottoproblema
```py
int computeParentesizzazione(int c[], int n){
    DP = new int [0...n][0...n]
    LAST = new int [0...n][0...n]
    for i=1 to n:
        DP[i][i] = 0                                                # la diagonale: moltiplico A[i] per A[i] (quindi nessun k)

    for h=2 to n:                                                   # h: indice diagonale   
        for i=1 to n-h+1:                                           # i: indice di riga
            int j = i+h-1                                           # j: indice di colonna
            DP[i][j] = INF

            for k=i to j-1:                                         # k: ultimo prodotto
                int tmp = DP[i][k] + DP[k+1][j] + c[i-1]*c[k]*c[j]  # provo a fare il prodotto
                if tmp < DP[i][j]:                                  # salvo il prodotto solo se è minore di quel che ho gia
                    DP[i][j] = tmp
                    LAST[i][j] = k

    return DP[1][n]
}
```


## INSIEME INDIPENDENTE DI INTERVALLI PESATI - dato un insieme di intervalli, devo trovare un sottoinsieme indipendente per massimizzare il profitto
> prendo il massimo tra non prendere e passare al prossimo (DP[i-1]) e prendere questo e il predecessore (DP[pred_i] + valore_i)
```py
int[] maxInterval(int a[], int b[], int valore[], int n){
    # ordino gli intervalli per estremi di fine crescenti
    int pred[] = computePred(a, b, n)
    int DP[] = new int[0...n]
    DP[0] = 0
    for i=0 to n:
        DP[i] = max(DP[i-1], valore[i] + DP[pred[i]])
    
    i=n
    int S[] = SET()
    while i>0:
        if DP[i-1]>valore[i]+DP[pred[i]]:
            i-=1
        else:
            S.insert(i)
            i = pred[i]
    return S
}
```


## MASSIMA SOTTOSEQUENZA CRESCENTE
```py
int longestIncreasing(int V[], int n){
    int DP[] = new int[0...n]
    int _max = 1

    for i=0 to n:
        DP[i] = 1           # se non trovo altro, questo elemento forma una sottoseq lunga 1
        for j=1 to i-1:
            if V[j]<V[i] and DP[j]+1 > DP[i]:
                DP[i] = DP[j] + 1
                if DP[i] > _max:
                    _max = DP[i]
    
    return _max
    # return max(DP)
}
```


## DONALD - Devo scegliere un sottoinsieme di citta che massimizzi il numero di elettori, tenendo conto di una distanza minima tra le citta
> DP[i] è il massimo numero di elettori che posso avere nelle prime i città. La soluzione è in DP[n]
```py
int donald(int miglia[], int elett[], int n, int minD){
    int DP[] = new int[0...n]
    DP[0] = elett[0]
    for i=1 to n:
        DP[i] = DP[i-1]                             # intanto pongo che in questa citta non passo (stessi elettori della precedente) [non prendo]
        int j=0
        while j<i and miglia[i]-miglia[j] >= minD:  # se trovo una citta precedente che rispetta la dist minima 
            DP[i] = max(DP[i], elett[i] + DP[j])    # vedo se mi conviene prendere quella e sommare quella attuale [prendo]
            i += 1
    return DP[n]
}
```


## BATTERIE - minimizzare il numero di fermate per completare il viaggio
> ogni fermata ha un costo e una distanza; c'è una distanzaMin dopo cui per forza devo fermarmi
> assumo che esistano due fermate fittizzie: 0 (con D[0]=0, C[0]=0) e n+1 (con D[n+1]=L, C[n+1]=0)
> DP[i] rappresenta il costo minimo acquistando la batteria alla stazione i. Sol: DP[0] perche parto dal fondo
```py
int minStops(int D[], int C[], int n, int r){
    int DP[] = new int[0...n+1]
    DP[n+1] = 0

    for i=n to 0:                               # parto dal fondo
        int j=i+1
        DP[i] = +INF
        while j <= n+1 and D[j] <= D[i]+r:      # cerco tra tutte le stazioni gia passate
            DP[i] = min(DP[i], DP[j])           # se ne trovo una a cui posso arrivare da i (la distanza tra j e i è <= r)
            j += 1                              # prendo il minimo tra le due
        DP[i] = DP[i] + C[i]                    # sommo il costo corrente
    
    return DP[0]
}
```


## COME PASSARE L'ESAME - minimizzzare il tempo impiegato per fare un totale di esercizi tc la somma dei valori sia V
> se non ho esercizi e devo ottenere v>0, ci metterò +INF
> se devo ottenere v=0, ci metto 0 perche non devo fare esercizi
```py
int studentePigro(int v[], int t[], int n, int V){
    int DP[][] = new int[0...n][0...V]
    for v=0 to V:
        DP[0][v] = INF
    
    for i=0 to n:
        DP[i][v] = min( DP[i-1][v],                         # non faccio l'esercizio, non cambia il valore, passo al precedente
                t[i] + (V-v[i]>0) ? DP[i-1][V-v[i]] : 0 )   # faccio l'esercizio e sommo il tempo al tempo che ci ho messo a fare 
                                                            # l'ultimo (se c'è, orelse 0)
}
```

## NUMERO DI ALBERI BINARI - calcolare il numero di alberi binari strutturalmente diversi (e k limitati)
```py
int kLimitato(int n, int k){
    int DP[][] = new int[0...n][0...k] = {-1}
    kLimitatoR(DP, n, k)
    return DP[n][k]
}

int kLimitatoR(int DP[][], int n, int k){
    if k<0:
        return 0    # ho cercato di costruire un albero troppo profondo
    elif n<=1:
        return 1    # ho un albero
    else:
        if DP[n][k] < 0:
            DP[n][k] = 0
            for i=0 to n-1:
                DP[n][k] = DP[n][k] + kLimitatoR(DP, i, k-1) * kLimitatoR(DP, n-1, i-1, k-1)
    return DP[n][k]
}
```


## MASSIMIZZARE NUMERO DI MONETE PER PAGARE T - problema del resto
```py
int maxResto(int V[], int i, int t, int DP[][]){
    if i==0 and t>0:
        return -INF         # ho ancora resto da dare, ma non ho piu monete
    if t<0:
        return -INF         # ho un resto negativo
    if t==0:
        return 0            # ho finito
    if DP[i][t] == -1:      # scelgo il max tra skippare la moneta o prenderla e sottrarre il valore che rimane
        DP[i][t] = max(maxResto(V, i-1, t, DP), maxResto(V, i-1, t-V[i], DP)
    return DP[i][t]
    
# Se dovessi contare il numero di modi in cui pagare invece
int resto(int V[], int i, int t, int DP[][]){
    if t==0:
        return 1
    if t<0 or i==0:
        return 0
    if DP[i][t] == -1:
        DP[i][t] = resto(V, i-1, t, DP) + resto(V, i, t-V[i], DP)
    return DP[i][t]
}
```



<br><hr><br>


# === Backtracking ===
## PRIMA ELEMENTARE - stampare tutte le possibili combinazioni date n palline gialle ed m palline rosse
```py
enumerate(char scelte[], int i, int gialleLeft, int rosseLeft){
    if gialleLeft==0 and rosseLeft==0:
        print scelte
    
    else:
        if gialleLeft > 0:
            scelte[i] = "G"
            enumerate(scelte, i+1, gialleLeft--, rosseLeft)
        if rosseLeft > 0:
            scelte[i] = "R"
            enumerate(scelte, i+1, gialleLeft, rosseLeft--)
}

solve(){
    char scelte[] = new char[0...n+m]
    enumerate(scelte, 0, n, m)
}
```
