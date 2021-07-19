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


## SOTTOSTRINGA PALINDROMA MASSIMALE
> DP[i][j] è la lunghezza della piu lunga sottostringa palindroma contenuta in S[i...j]
> notare che j-i+1 è la lunghezza della sottostringa
```py
maxPalSubs(int s[], int n, int i, int j, int DP[][]{
    if j-i+1 <= 1:
        return j-i+1
    if s[i] == s[j] and maxPalSubs(s, n, i+1, j-1, DP) == j-i+1:    # se 0 ed n sono uguali e in mezzo ho una palindroma, allora è palindroma
        return j-i+1
    return max( maxPalSubs(s, n, i+1, j, DP), maxPalSubs(s, n, i, j-1, DP) )    # else prendo la subs massimale scegliendo tra levare un char a dx o sx
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


## LUNGHEZZA DELLA MASSIMA SOTTOSEQUENZA CRESCENTE DISTINTA - data una stringa
> ![image](https://user-images.githubusercontent.com/33253698/126119749-83372ac8-cbe8-422c-958c-49a971da3fd6.png)
> Tuttavia, il modo piu semplice per risolvere questo problema è rendersi conto che è sufficiente utilizzare l’algoritmo LCS, chiamato
passando questa stringa e la stringa ABCDEFGHIJKLMNOPQRSTUVWXYZ. Il costo di questa soluzione è pari a Θ(n), in quanto deve
essere riempita una matrice n × 26.
```py
int maxOrdinataDistinta(ITEM[ ] S, int n){
    int DP[][] = new int[0...n][0...26]
    lcs(M, S, ”ABCDEFGHIJKLMNOPQRSTUVWXYZ”, n, 26)
    return DP[m, n]
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

# Se avessi un numero illimitato di monete, e dovessi calcolare se posso dare resto R con massimo k monete
int limRemainder(int v[], int n, int R, int k){
    int DP[][] = new int[0...n][0...R][0...k] = {-1}
    return limRec(v, n, R, k, DP)
}

int limRec(int v[], int i, int r, int j, int DP[][]){
    if r<0:
        return false
    if r==0:
        return true
    if i==0 or j==0:
        return false
    if DP[i][r][j] == -1:
        DP[i][r][j] = limRec(v, i, r-v[i], j--, DP) or limRec(v, i-1, r, j, DP)
    return DP[i][r][j]
}
```


## SCACCHIERA - numero di percorsi distinti da 0,0 a m,n (0/1 se attraversabile o meno)
```py
int countPercorsi(int M[][], int m, int n){
    int DP[][] = new int[0...m][0...n]
    DP[m][n] = M[m][n]
    for i=m-1 to 0:
        DP[i][n] = M[i][n]*DP[i+1][n]
    for j=n-1 to 0:
        DP[m][j] = M[m][j]*DP[m][j+1]
    for i=m-1 to 0:
        for j=n-1 to 0:
            DP[i][j] = M[i][j]*(DP[i][j+1] + DP[i+1][j])
    return DP[0][0]
}
```


## MIN MOVES - numero minimo di mosse in un vettore per andare da 0 a n (ogni casella contiene il max di passi possibili da lì)
```py
minMoves(int V[], int n){
    int S[] = new int[0...n]
    S[n] = 0                    # andare da n ad n mi costa 0
    for i=n-1 to 0:
        S[i] = +INF             # effettuo una mossa e mi metto nella posizione che
        for k=0 to V[i]:        # poi richede il minor numero di mosse
            if i+k <= n and S[i+k]+1 < S[i]:
                S[i] = S[i+k] + 1
    return S[n]
}
```


## DADI - numero di modi diversi con cui ottenere una certa somma
![image](https://user-images.githubusercontent.com/33253698/126070133-a1303e79-83ce-45bd-82f1-708d219f5a07.png)
```py
    404 code not found
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


## COLORING - si può colorare un grafo non orientato con al max k colori, in modo che ogni nodo sia diverso dagli adiacenti?
```py
bool coloring(Graph G, int k, int S[], int u){
    set C = {1...k}
    foreach v in u.adj():
        if S[v] != 0:
            C.remove(S[v])
    foreach c in C:
        S[u] = c
        if u==G.n:
            printSol(S, G.n)
            return true
        else
            if coloring(G, k, S, u+1):
                return true
        S[u] = 0
    return false
}
```


<br><hr><br>


# === Flusso ===
## BALLO DI FINE ANNO
![image](https://user-images.githubusercontent.com/33253698/126070889-9ec41c2b-97fe-4b78-8000-f6b60652ebf8.png)
![image](https://user-images.githubusercontent.com/33253698/126070903-0abc832e-9da0-4228-84ca-faaf4a09f199.png)
![image](https://user-images.githubusercontent.com/33253698/126070913-47a9afd0-31f0-48da-b047-d1d37010d6f6.png)


## REGOLAMENO DEL DISI
![Schermata da 2021-07-19 09-17-06](https://user-images.githubusercontent.com/33253698/126118660-1c97c735-fb2c-46a5-9696-3feb0f8e961b.png)
![Schermata da 2021-07-19 09-18-19](https://user-images.githubusercontent.com/33253698/126118759-60a70170-23bb-4246-b349-619327c96790.png)


## WORKSHOPs
![image](https://user-images.githubusercontent.com/33253698/126140513-f6962a88-a8b7-4487-8662-5c1b6dae3018.png)
![image](https://user-images.githubusercontent.com/33253698/126140566-6bcac8bf-f58e-4bdd-8a9b-9148c029a782.png)
![image](https://user-images.githubusercontent.com/33253698/126140620-816fe820-f86a-427e-bcba-e28d3e28cb0c.png)

