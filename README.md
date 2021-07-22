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

<br>

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

# C'è anche una versione `bits(int n)` in cui bisogna contare il numero di sringhe binarie ottenibili di n bit
# che non contengono bit 1 consecutivi. Il procedimento è lo stesso, ma devo sommare e non prendere il max
```

<br>

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


<br>

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


<br>

## 3DKNAPSACK - matrice a tre dimensioni aka con un constraint in piu
> trovare il valore della somma massimale (k,w)-vincolata, cioè il piu grande valore ottenibile
> come somma di k valori tale che sia minore/uguale a w; DP[i][s][v] è il massimo valore ottenibile
> usando i primi i elementi, scegliendo al massimo s valori, non dovendo superare v
Complex: $O(nkw)$
```py
int kwConstraint(int X[], int n, int k, int w){
    int DP[][][] = new int [0...n][0...k][0...w]
    for i=0 to n:
        for s=0 to k:
            for v=0 to w:
                DP[i][s][v] = -1            # init
    return kwConstraintRec(X, n, k, w, DP)
}

int kwConstraintRec(int X[], int i, int s, int v, int DP[][][]){
    if i==0 or s==0 or v==0:
        return 0                # se non ho piu elementi da/che posso scegliere o v è finito, ho 0 modi
    if DP[i][s][v]<0:
        if X[i]>v:              # se il numero è troppo grande non posso sommarlo, non lo prendo
            DP[i][s][v] = kwConstraintRec(X, i--, s, v, DP) 
        else:
            DP[i][s][v] = max(
                kwConstraintRec(X, i--, s, v, DP),
                kwConstraintRec(X, i--, s--, v-X[i], DP)+X[i],  # prendo: levo il valore dalla somma, 
            )                                                   # diminuisco le scelte possibili e sommo il valore
    return DP[i][s][v]
}
```


<br>

## SOTTOSEQUENZA COMUNE MASSIMALE - date due sequenze T ed U, quanto sono simili tra loro?
Complex: $O(nm)$
```py
# in caso di una sottostringa comune massimale (contigua), guardare i commenti, che si riferiscono alle righe precedenti il commento
int lcs(int T[], int U[], int n, int m){
    DP = new int[0...n][0...m]
    for i=0 to n:
        DP[i][0] = 0
    for j=0 to m:
        DP[0][j] = 0
    # -4 int maxSoFar = 0

    for i=1 to n:
        for j=1 to m:
            if T[i] == S[j]:
                DP[i][j] = DP[i-1][j-1] + 1   # se gli ultimi caratteri sono uguali, calcolo la scm snza contarli e sommo 1 (length)
                # + maxSoFar = max(maxSoFar, DP[i][j])
            else:
                DP[i][j] = max( DP[i-1][j], DP[i][j-1] ) # se son diversi, calcolo la migliore che avrei togliendo l'ultimo carattere ad ognuno
                # - DP[i][j] = 0
    return DP[n][m]
    # - return maxSoFar
}
```


<br>

## MIN REMOVE - minimo numero di elementi da rimuovere da un vettore per far si che max(A)-min(A) <= k
Complex: $O(n)$
```py
# Approcio Greedy
int minRemove(int A[], int n, int k){
    sort(A, n)
    int i=0
    int j=0
    int maxSoFar = 0
    while i<=n and j<=n:
        if A[j] - A[i] <= k:
            maxSoFar = max(maxSoFar, j-i+1)
            j += 1
        else:
            i += 1
    return n - maxSoFar
}
```


<br>

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


<br>

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


<br>

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


<br>

## INSIEME INDIPENDENTE DI INTERVALLI PESATI - dato un insieme di intervalli, devo trovare un sottoinsieme indipendente per massimizzare il profitto
> prendo il massimo tra non prendere e passare al prossimo (DP[i-1]) e prendere questo e il predecessore (DP[pred_i] + valore_i)
```py
int[] maxInterval(int a[], int b[], int valore[], int n){
    # ordino gli intervalli per estremi di fine crescenti: O(nlogn)
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

# O(n^2), ma ne esiste una versione nlogn
computePredecessor(int a[], int b[], int n){
    int pred[] = new int[0...n]
    p[0] = 0
    for i=1 to n:
        j = i-1
        while j>0 and b[j] > a[i]:
            j--
        pred[i] = j
    return pred
}
```


<br>

## MASSIMA SOTTOSEQUENZA CRESCENTE
Complex: $O(n^2)$
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
                # DP[i] = max(DP[i], DP[j]+1)
    
    return _max
    # return max(DP)
}
```


<br>

## SEQUENZA K-LIMITATA MASSIMALE
Complex: $O(n^2)$
```py
int ksequence(int A[], int n, int k){
    int DP[] = new int[0...n]
    for i=0 to n:
        DP[i] = A[i]
        for j=0 to i-1:
            if |A[j]-A[i]| <= k and DP[j] + A[i] > DP[i]:
                DP[i] = DP[j] + A[i]
    return max(DP)
}
```


<br>

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


<br>

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


<br>

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


<br>

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

<br>

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


<br>

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


<br>

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


<br>

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


<br>

## DADI - numero di modi diversi con cui ottenere una certa somma
![image](https://user-images.githubusercontent.com/33253698/126070133-a1303e79-83ce-45bd-82f1-708d219f5a07.png)
```py
    404 code not found
```


<br>

## MONTRESOR INVESTMENT COMPANY - massimizzare il profitto comprando e vendendo azioni, sapendo i prezzi di n giorni
![image](https://user-images.githubusercontent.com/33253698/126392866-e5a2c9ce-9d29-464a-a8fc-cec0be5dd31c.png)


<br>

## NUMERO DI MODI PER RAPPRESENTARE UNA STRINGA COME SOMMATORIA
Complex: $O(n^4)$ perche ho una tabella potenzialmente $n\times n$, e per riempire ogni cella ho $O(n^2)$
```py
int countSum(int V[], int k){
    int DP[][] = new int[0...n][0...k]
    for i=0 to n:
        for r=0 to k:
            DP[i][r] = -1
    return countRec(V, n, k, DP)
}

int countRec(int V[], int i, int r, int DP[][]){
    if i==0 and r==0:                       # 1 modo per ottenere 0 
        return 1                            # non avendo termini da sommare
    if (i>0 and r<=0) or (i==0 and r>0):    # 0 modi per ottenere negativo/nullo sommando positivi
        return 0                            # 0 modi per ottenere positivo se non ho termini da sommare

    if DP[i][r] == -1:                      # spezzo le i cifre rimanenti in due parti (con s)
        DP[i][r] = 0                        # e applico ricors la sommtoria da 0 a s-1
        for s=0 to i:                       
            DP[i][r] = DP[i][r] + countRec(V, s--, r-value(V, s, i), DP)
    return DP[i][r]
}
```


<br>

## OTTENERE K COME SOMMA DEGLI ELEMENTI DI UN VETTORE A
Complex: $O(nr)$
```py
int countMenu(int A[], int n, int k){
    int DP[] = new int[0...n][0...k] = {-1}
    return countRec(A, n, k, DP)
}

int countRec(int A[], int i, int r, int DP[]){
    if r==0:
        return 1            # per ottenere 0 ho un solo modo: non selezionare elementi
    elif i==0:
        return 0            # se non ho elementi da sommare, non posso ottenere r
    else:
        if DP[i][r] == -1:
            DP[i][r] = countRec(A, i--, r, DP)
            if A[i] <= r:
                DP[i][r] = DP[i][r] + countRec(A, i--, r-A[i], DP)
        return DP[i][r]
}  
```


<br>

## VALORE DEL SOTTOVETTORE DI LUNGHEZZA PARI DI SOMMA MASSIMALE
Complex: $\Theta(n)$
```py
int maxSumEven(int A[], int n){
    int DP[] = new int[0...n]
    DP[0] = DP[1] = 0
    for i=2 to n:
        DP[i] = max(DP[i-2]+A[i-1]+A[i], 0)
    return max(DP, n)
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


<br>

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


<br>

## PRINT SUMS - stampare tutti i modi in cui posso ottenere w sommando i valori di X[]
Complex: $O(n\cdot 2^n)$
```py
void printSums(int S[], int X[], int i, int n, int w){
    if i==n:                                        # se sono arrivato a 0 e ho finito
        if v==0:                                    # gli elementi, stampo
            print(S)
    else:
        for c in [0, 1]:
            S[i] = c                                # prendo/non prendo
            printSums(S, X, i++, n, w-c*X[i])       # se moltiplico per 0 non sottraggo
}
```


<br>

## DOVE COMPARE IL "+"? - data una stringa di interi, dire le possibili combinazioni come somme
Complex: $O(n\cdot 2^n)$ (ho $2^{n-1}$ possibili combinazioni, piu il tempo per stamparle aka $O(n)$)
```py
printAllRec(int S[], int n, bool stop[], int i){
    if i==0:
        print(S[0])
        for i=1 to n:
            if stop[i]:
                print("+")
            print(S[i])
        println()
    else:
        for c in [True, False]:
            stop[i] = c
            printAllRec(S, n, stop, i-1)
}
```


<br>

## CAMMINI LUNGHI K IN UN GRAFO - stampare tutti i cammini lunghi k a partire da un nodo s
```py
visit(Graph G, int k, int s){
    bool visited[] = new bool[0...G.length] = {False}
    int path[] = new int[0...k]
    visitRec(G, k, s, 0, path, visited)
}

visitRec(Graph G, int k, Node u, int i, int path[], bool visited[]){
    path[i] = u
    if i==k:
        print path
    else:
        visited[u] = True
        for v in G.adj(u):
            if not visited[v]:
                visitRec(G, k, i++, v, path, visited)
        visited[u] = False

}
```


<br>

## LONGEST PATH - lunghezza del percorso piu lungo formato da 1 che inizia in 1,1
Complex: $O(4^{nm})$
```py
int longestPath(int M[][], int n, int m){
    return longestPathRec(M, n, m, 1, 1)
}

int longestPathRec(int M[][], int n, int m, int i, int j){
    if 1<=i<=n and 1<=j<=m and M[i][j] == 1:
        M[i][j] = -1            # come se mettessi True su visited
        int _max = 1 + max(
                        longestPathRec(M, n, m, i+1, j)                            
                        longestPathRec(M, n, m, i-1, j)                            
                        longestPathRec(M, n, m, i, j+1)                            
                        longestPathRec(M, n, m, i, j-1)
        )
        M[i][j] = 1
        return _max
    else:
        return 0
}
```


<br>

## CICLI HAMILTONIANI IN UN GRAFO
```py
printHamilton(Graph G){
    bool visited[] = new bool[0...G.length] = {False}
    int path[] = new int[0...G.size]
    visitRec(G, 0, 0, path, visited)
}

visitRec(Graph G, Node u, int i, int path[], bool visited[]){
    path[i] = u
    if i==G.size:
        print path
    else:
        visited[u] = True
        for v in G.adj(u):
            if not visited[v]:
                visitRec(G, v, i++, path, visited)
        visited[u] = False

}
```


<br>

## STAMPARE TUTTE LE SEQUENZE DI INDICI DI T CHE GENERANO UN PATTERN P
Complex: $O(2^n)$
```py
printAll(item T[], item P[], int i, int j, Stack S){        
    if j==0:
        print S
    else if i>= j:
        printAll(T, P, i--, j, S)       # ignoro l'ultimo carattere

        if T[i] == P[j]:                # prendo l'ultimo carattere
            S.push(i)
            printAll(T, P, i--, j--, S)
            S.pop()
}
```


<br>

## STAMPARE TUTTE LE SOTTOSEQUENZE CRESCENTI DI UN VETTORE
Complex: $O(n\cdot 2^n)$
```py
printIncreasing(int A[], int n){
    Stack S = Stack()
    printRec(A, n, S)
}

printRec(int A[], int i, Stack S){
    if i==0:
        print S
    else:
        if S.empty() or A[i] < S.pop():
            S.push(A[i])
            printRec(A, i--, S)
            S.pop()
        printRec(A, i--, S)
}
```


<br>

## STAMPARE TUTTE LE SOMME PALINDROME DI N
Complex: $O(n\cdot 2^{n/2}$) dove $O(n)$ è per la stampa
```py
genRec(int S[], int i, int missing){
    for k=0 to i-1:
        print(S[k])
    if missing>0:
        print missing
    for k=i-1 to 0:
        print(S[k])
    for j=0 to missing/2:
        S[i] =j
        genRec(S, i++, missing-2*j)
}

generate(int n){
    int S[] = new int[0...n/2]
    genRec(S, 0, n)
}
```


<br>

## STAMPARE TUTTE LE POSSIBILI PARENTESIZZAZIONI COMPOSTE DA N COPPIE DI PARENTESI
Complex: $\Omega(ncdot 2^n)$
```py
printPar(int n){
    Item S[] = new Item[2*n]
    printRec(S, 0, n, n, 0) 
}

printRec(int S[], int open, int toOpen, int toClose, int i){
    if toOpen == 0 and toClose == 0:
        print S
    else:
        if toOpen > 0:
            S[i] = "("
            printRec(S, open+1, toOpen-1, toClose, i+1)
        if toClose > 0:
            S[i] = ")"
            printRec(S, open-1, toOpen, toClose-1, i+1)
}
```


<br><hr><br>


# === Flusso ===
## FORD FULKERSON
```py
int[][] maxFlox(Graph G, Node S, Node T, int c[][]){
    int f[][] = new int [][]                # Flusso parziale
    int g[][] = new int [][]                # Flusso da cammino aumentante
    int r[][] = new int [][]                # Rete residua

    for x,y in G.V():
        f[x][y] = 0                         # Inizializza un flusso nullo
        r[x][y] = c[x][y]                   # Copia c (capacita?) in r
    
    repeat:
        g = flusso associato ad un cammino aumentante in r (oppure f di 0)
        for x,y in G.V():
            f[x][y] = f[x][y] + g[x][y]     # f = f+g
            r[x][y] = c[x][y] - f[x][y]     # Calcola c di f
    until g==f di 0
    return f
}
```


<br>

## BALLO DI FINE ANNO
![image](https://user-images.githubusercontent.com/33253698/126070889-9ec41c2b-97fe-4b78-8000-f6b60652ebf8.png)
![image](https://user-images.githubusercontent.com/33253698/126070903-0abc832e-9da0-4228-84ca-faaf4a09f199.png)
![image](https://user-images.githubusercontent.com/33253698/126070913-47a9afd0-31f0-48da-b047-d1d37010d6f6.png)


<br>

## REGOLAMENO DEL DISI
![Schermata da 2021-07-19 09-17-06](https://user-images.githubusercontent.com/33253698/126118660-1c97c735-fb2c-46a5-9696-3feb0f8e961b.png)
![Schermata da 2021-07-19 09-18-19](https://user-images.githubusercontent.com/33253698/126118759-60a70170-23bb-4246-b349-619327c96790.png)


<br>

## WORKSHOPs
![image](https://user-images.githubusercontent.com/33253698/126140513-f6962a88-a8b7-4487-8662-5c1b6dae3018.png)
![image](https://user-images.githubusercontent.com/33253698/126140566-6bcac8bf-f58e-4bdd-8a9b-9148c029a782.png)
![image](https://user-images.githubusercontent.com/33253698/126140620-816fe820-f86a-427e-bcba-e28d3e28cb0c.png)


<br>

## HATEVILLE
![image](https://user-images.githubusercontent.com/33253698/126554314-839ddfa9-7e45-46e0-a8a7-819937d19ac6.png)



<br><hr><br>


# === Cammini Minimi ===
## Problema
> Un **cammino** p è un insieme (ordinato?) di nodi $v_1...v_k$. Il peso di quel cammino è la somma dei costi degli archi che lo compongono.
Il cammino minimo da _s_ ad _u_ è un cammino il cui costo è minore o uguale di qualsiasi altro cammino tra gli stessi nodi.

> Un **albero di copertura** di G è un sottografo tale che
> - sia un albero
> - i suoi archi siano sottoinsieme degli archi di G
> - contenga tutti i vertici di G

<br>

## Cammini minimi a sorgente *singola*
### Teorema di Bellman
> Una soluzione è **ammissibile** quando può essere descritta da 
> - un albero di copertura radicato in s 
> - un vettore di distanza

> Una soluzione ammissibile $T$ è **ottima** se e solo se
> - per ogni arco $(u,v)\in T$, la distanza di v è uguale alla distanza di u + il peso di $(u,v)$
> - per ogni arco $(u,v)\in E$, la distanza di v è minore/uguale alla distanza di u + il peso di $(u,v)$

<br>

### Algoritmo generico
- Ho una struttura dati a cui aggiungo i nodi man mano, e dei booleani in cui segno quali nodi sono nella struttura dati
- Finche la struttura non è vuota, estraggo uno ad uno i nodi e itero nei suoi adiacenti
- Se la distanza all'adiacente v è minore dell'attuale (inizialmente +INF) e non è nella struttura, la aggiorno e lo inserisco. (e segno il padre)
- Se è già nella struttura... dipende dall'algoritmo

<br>

### Dijkstra
Funziona bene solo con pesi positivi. Sfrutta le code con priorità (inzialmente basate su vettori, con costo O(n)).
Estraggo il nodo con priorità minima e cancello la sua priorità

Ho complessità O(n^2)
IMMAGINE PAG 27

<br>

### Johnson
Code con priorità, basate però su heap binario che riduce il costo a O(m logn) (COS'È M?)

<br>

### Fredman-Tarjan
Code con priorità, basate però su heap di Fibonacci che riduce il costo *ammortizzato* a O(m + nlogn) (COS'È M?)

<br>

### Bellman-Ford-Moore
È più pesante di Dijkstra, ma funziona con archi di peso negativo. Utilizzo una coda. Non faccio nulla nel caso in cui il nodo sia già nella coda. Ha complessità O(nm) perche ogni nodo puo essere estratto massimo n-1 volte.

IMMAGINE PAG 37

<br>

### DAG
Poiche non esistono cicli non rischio di tornare su un nodo gia visitato e abbassare il valore della sua distanza. Posso utilizzare l'ordinamento topologico

IMMAGINE PAG 39


<br>

## Cammini minimi a *sorgente multipla*
Potrei ripetere Dijkstra o Johnson o Bellman Fork (per grafi densi o sparsi o con peso negativo) aumentando di un fattore n la complessità.
Altrimenti posso usare FloydWarshall o Johnson per sorgente multipla

> Un cammino tra x e y è *k-vincolato* se non passa per nessun vertice v>k.

> La distanza *k-vincolata* tra x e y è il costo totale del cammino minimo k-vincolato tra x e y (se esiste, altrimenti è +INF)
