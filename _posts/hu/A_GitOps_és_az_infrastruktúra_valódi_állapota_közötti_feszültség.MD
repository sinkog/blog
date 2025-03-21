# A GitOps és az infrastruktúra valódi állapota közötti feszültség

## Bevezetés

A **GitOps** mára egy széles körben elfogadott megközelítés a modern infrastruktúra-menedzsmentben. Lényege, hogy az **elvárt állapotot** (desired state) deklaratívan, verziókezelten, Git-ben tároljuk, majd automatizált rendszerek gondoskodnak arról, hogy ez az állapot **megegyezzen a valós rendszer állapotával**.

Egy ideális GitOps-folyamat 5 lépésből áll:

1. **Elvárt állapot deklarálása** (YAML/JSON formában Git-ben)
2. **Diff számítása** (mi különbözik a valóságtól?)
3. **Validáció** (a változás végrehajtható-e biztonságosan?)
4. **Végrehajtás** (infrastruktúra változás implementálása)
5. **Ellenőrzés és visszacsatolás** (az új állapot valóban az elvárt lett?)

Ezek után az állapot **folyamatosan figyelve** van, és bármilyen „drift” (elsodródás) esetén újraindítható a szinkronizációs folyamat.

## A probléma: az állapot hol van és kiben bízunk?

GitOps modellben az *elvárt állapot* adott.  
De az *aktuális állapot* forrása nem triviális:

- Kubernetes-ben ez egyszerű: a `kubectl get` bármikor lekéri az állapotot, CRD-k, controller-ek automatikusan reconciliálnak.
- De mi van, ha Proxmoxot, bare-metal gépeket, vagy akár egy AWS EC2 példányt kezelünk Terraformmal?

Itt lép be a probléma: ezeknél az eszközöknél **nem egyértelmű, ki tárolja a „valóságot”**.

## Példa: Terraform és az állapot tárolása

A Terraform például `.tfstate` fájlban tárolja, **amit ő hisz**, hogy a valós rendszerben van.  
Ez a `tfstate` viszont:

- nem automatikusan frissül
- nem mindig egyezik a valósággal
- *nem is szinkronizálódik*, csak ha parancsot adunk rá (`refresh`, `import`, stb.)

Ez azt eredményezi, hogy **a „diff” számítás nem a valódi valóság alapján történik**, hanem egy cached, lokális állapotképen – ami idővel biztosan **eltér** a rendszer valós állapotától.

## Mit jelent ez GitOps szemléletből?

GitOps egyetlen forrásnak tekinti a Git-ben tárolt specifikációt.  
Ezért elvárja, hogy:

- legyen egy megbízható `diff` a valóság és a Git között
- ez a `diff` determinisztikus és auditálható legyen
- a végrehajtás **csak akkor** történjen, ha szükséges
- a visszacsatolás biztosítsa a konzisztenciát

Ez csak akkor működik, ha:

✅ van **lekérhető, gépi formában elérhető rendszerállapot**  
✅ a rendszer képes **értelmes diff** előállítására  
✅ a **végrehajtás idempotens** és visszaellenőrizhető

## Ki alkalmas erre?

| Eszköz / Rendszer             | Alkalmas? | Megjegyzés                                                       |
|------------------------------|-----------|-------------------------------------------------------------------|
| **Kubernetes**               | ✅        | Beépített `state`, reconciliation loop, CRD támogatás             |
| **Ansible**                  | ⚠️        | Pull-alapú, de nincs állapot; csak „feltételez” (`when`, `stat`) |
| **Terraform**                | ❌/⚠️     | Lokális vagy külső `state`, ami nem frissül automatikusan        |
| **Pulumi**                   | ⚠️        | Jobb mint Terraform, de még mindig saját `state`                  |
| **Saját API-figyelő rendszer** | ✅        | Observer + saját `state` összevetés → teljes kontroll             |

## Mi lenne az ideális?

A GitOps szellemiségéhez az illik leginkább, ha:

- A valós állapot **pull-alapúan** kérdezhető (nem a változtatás írja felül)
- A rendszer képes normalizált, objektum-alapú `state` előállítására
- Ez a `state` **verziózható**, összehasonlítható a `spec`-tel
- A változás **modulárisan végrehajtható** (API, Terraform, Shell, bármi)

Ez az irány nyitja meg az utat **terraform-független, központosított állapotvezérelt rendszerek** felé – ahol a hangsúly **nem az eszközön, hanem a logikán** van.

## Zárás

A GitOps nem egy eszközhasználati módszer, hanem **egy szemlélet**:
> *a rendszer állapotát deklaráljuk, figyeljük, és csak akkor avatkozunk be, ha kell.*

E szemlélet akkor valósítható meg igazán, ha:

- a rendszer **kiszámíthatóan modellezhető**
- az állapot **valóban lekérdezhető**
- a változás **ellenőrizhető és reprodukálható**

És ezek nem eszközfüggő követelmények — hanem **architekturális elvek**, amelyekre eszközöket lehet választani. De ha egy eszköz nem képes ezeknek megfelelni (pl. Terraform `tfstate`), akkor **nem illik a GitOps modellbe — legalábbis nem vezető szerepben.**
