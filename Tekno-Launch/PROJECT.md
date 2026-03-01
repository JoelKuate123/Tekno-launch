# Auto-RAG : Audit Intelligent de Documents Techniques Sensibles

## Projet TEKNO-LAUNCH 2026 — Track AI & Cloud

---

## 1. Vision du Projet

### Objectif
Developper une IA capable d'analyser automatiquement ~10 000 pages de documents techniques sensibles (Safety Cases nucleaires) pour :
- **Detecter** erreurs, incoherences et contradictions entre documents
- **Proposer** des corrections pertinentes avec 0% d'approximation
- **Cartographier** les impacts transversaux (si une correction est appliquee, quels autres documents sont affectes ?)

### Ce que ce N'EST PAS
Ce n'est **pas** un chatbot ou un assistant Q&A. C'est un **outil d'audit documentaire intelligent** avec analyse d'impact.

### Client
**TEKNO-SAFETY** — Organisme de gestion des dechets radioactifs du Benelux (Belgique, Pays-Bas, Luxembourg). Gere les Safety Cases pour le stockage geologique profond dans l'argile de Boom.

---

## 2. Contexte Metier

### Corpus Documentaire
| Type | Volume | Description |
|------|--------|-------------|
| Safety Cases/Reports | ~10 000 pages | Documents de surete nucleaire |
| Notes RD&D | 300+ | Recherche, Developpement & Demonstration |
| Standards | IAEA SSR-5, SSG-23, SSG-14 | Normes internationales |
| Regulateur | AFCN/FANC (Belgique) | Cadre legal |

### Domaine Technique
- Stockage geologique profond de dechets HAVL (Haute Activite Vie Longue)
- Argile de Boom a -225m (Mol-Dessel, Belgique)
- Conteneurs acier S235, barriere bentonite MX-80
- Echelles de temps : jusqu'a 1 000 000 ans

### Problemes de l'Approche Manuelle
- **1000-1500h par dossier** de revue manuelle
- Fatigue humaine = erreurs
- 300+ documents interconnectes = goulot d'etranglement
- Tracabilite fragmentee
- Expertise rare et non scalable

---

## 3. Innovation : Auto-RAG

### RAG Classique vs Auto-RAG

| Aspect | RAG Classique | Auto-RAG |
|--------|--------------|----------|
| Direction | Unidirectionnel | **Bidirectionnel** |
| Objectif | Repondre a une question | **Detecter incoherences** |
| Input | Question externe | **Chaque chunk** |
| Output | Reponse + citations | **Rapport I0/I1/I2/I3** |
| Complexite | O(n) | **O(n²)** |

### Principe Cle
Chaque chunk joue un **double role** :
1. **Prompt de verification** : "Suis-je coherent avec les autres ?"
2. **Base de verite** : Reference pour verifier les autres chunks

Resultat : Verification croisee **NxN exhaustive**.

---

## 4. Typologie des Incoherences

| Niveau | Nom | Description | Exemple |
|--------|-----|-------------|---------|
| **I0** | Clean | Aucune incoherence detectee | Document valide |
| **I1** | Entre refs | Conflit entre 2 documents de reference | IAEA vs norme locale |
| **I2** | Doc vs Ref | Document contredit sa reference | Safety Case vs standard |
| **I3** | Auto-contradiction | Document se contredit lui-meme | Section 3 != Section 7 |
| **I4** | Multi-niveaux | Incoherences combinees consolidees | Cascade d'impacts |

### Documents Exemples Fournis
- `I0_clean_SC_corrosion_conteneurs.pdf` — Document de reference valide (corrosion conteneurs HAVL)
- `I1_entre_refs_transport_I129.pdf` — Conflit entre references sur le transport de l'Iode-129
- `I2_doc_vs_ref_PA_evaluation_performances.pdf` — Contradictions doc/reference sur l'evaluation de performances
- `I3_autocontradiction_THM_argile_boom.pdf` — Auto-contradictions sur le comportement thermo-hydro-mecanique
- `I4_multi_niveaux_SC_consolide_2024.pdf` — Incoherences multi-niveaux consolidees

---

## 5. Architecture Multi-Agents

### 4 Agents Specialises

```
                    +------------------+
                    |   ORCHESTRATEUR   |
                    +--------+---------+
                             |
          +------------------+------------------+
          |         |           |               |
    +-----+---+ +--+------+ +--+--------+ +----+------+
    |Agent RAG| |Agent     | |Agent      | |Agent      |
    |         | |Temporel  | |Semantique | |Causalite  |
    +---------+ +----------+ +-----------+ +-----------+
    Indexation   Coherence    Coherence     Cause-Effet
    & Retrieval  Chronologique Terminologie  Contradictions
    Chunking     Dates        NER           Logiques
    Embeddings   Sequences    Coreference
    Top-k        Causalite    Vocabulaire
```

#### Agent RAG
- Chunking semantique des documents
- Generation d'embeddings (BERT)
- Indexation dans base vectorielle (FAISS)
- Retrieval Top-k pour verification croisee

#### Agent Temporel
- Coherence chronologique entre documents
- Detection d'anachronismes
- Verification des sequences temporelles

#### Agent Semantique
- NER (Named Entity Recognition) sur le vocabulaire technique
- Coreference resolution
- Coherence terminologique entre documents

#### Agent Causalite
- Extraction relations cause-effet
- Verification des chaines logiques
- Detection de contradictions logiques (A ET non-A)
- Verification de transitivite

### Workflow d'Orchestration
1. **Init** : Charge corpus, distribue aux agents
2. **Parallelisation** : Agents travaillent simultanement
3. **Collecte** : Agregation des findings, deduplication
4. **Priorisation** : Scoring I1/I2/I3, precedence
5. **Rapport** : JSON + Dashboard + Actions

---

## 6. Strategie de Chunking

| Methode | Taille | Avantages | Inconvenients | Recommande ? |
|---------|--------|-----------|---------------|-------------|
| Fixed-Size | 256/512 tokens | Simple, rapide, faible cout | Coupe les idees, perd contexte | Non |
| Content-Aware | Par paragraphe | Coherent, bon compromis | Taille variable | Possible |
| **Semantic** | Clustering ML | Preserve le sens, optimal RAG | Cout calcul, setup complexe | **Oui** |

---

## 7. Contraintes Critiques

### Confidentialite (ZERO tolerance)
- Azure tenant prive TEKNO-SAFETY
- VNet isolation complete
- Private endpoints (zero internet)
- Chiffrement AES-256
- Azure Key Vault pour les secrets
- Modeles on-tenant (pas d'API publique)

### Protection Injections Prompt
- Input sanitization (patterns regex)
- Prompt templating strict
- Output validation format
- Scoring de confiance
- Sandbox isole
- Rate limiting
- Human-in-the-loop pour haute severite

### KPI Qualite

| Metrique | Seuil | Priorite | Methode |
|----------|-------|----------|---------|
| Hallucinations | < 10% | BLOQUANT | Validation humaine 100 chunks |
| Faux Positifs (Type I) | < 5% | HAUTE | Revue alertes vs verite terrain |
| Faux Negatifs (Type II) | < 3% | CRITIQUE | Corpus test annote |
| Precision globale | > 92% | HAUTE | Matrice confusion |

---

## 8. Roadmap (17 semaines)

### Phase 1 — POC (3 semaines)
**Objectif** : Faisabilite technique sur ~190 pages avec 4 agents

| Sprint | Duree | Activites | Livrables |
|--------|-------|-----------|-----------|
| S1 | 1 sem | Analyse corpus, vocabulaire technique, choix chunking, setup Azure | Architecture doc, Repo init |
| S2 | 1 sem | Pipeline chunking, embeddings BERT, base FAISS, tests unitaires | Agent RAG v0.1, Corpus indexe |
| S3 | 1 sem | Agent Temporel, Agent Semantique, Orchestrateur, Precedence docs | 4 agents, Auto-RAG loop |
| S4 | integre | Tests corpus 190p, mesure KPI, rapport incoherences, REX ONDRAF | POC valide, Rapport Phase 1 |

**Criteres de succes** :
- 4 agents operationnels
- Auto-RAG fonctionnel sur corpus test
- KPI atteints (< 10% hallucinations, < 5% FP, < 3% FN)
- Rapport auto-genere
- Temps < 2h pour 200 pages
- Validation ONDRAF positive

### Phase 2 — Scale-Up (3+ semaines)
**Objectif** : Passage a >1000 pages + sources heterogenes + DB externes

- Optimisation pipeline (1 sem)
- Multi-format ingestion PDF/DOCX/Excel (1 sem)
- Integration DB SQL/NoSQL (x sem)
- Tests performance (1 sem)
- Tuning KPI (2 sem)

### Phase 3 — Infrastructure Cloud (6 semaines)
**Objectif** : Architecture Azure privee production-ready

```
+-------------------+
|    Front-end      |  Interface Q&A + Dashboard
+-------------------+
|   API Gateway     |  FastAPI + Auth
+-------------------+
|  Orchestrateur    |  Multi-Agent Coordination (Azure App Container)
+-------------------+
|  Services Core    |  RAG + Agents
+-------------------+
|   Data Layer      |  Vector DB (self-hosted) + Cache + Storage
+-------------------+
|  Infrastructure   |  Azure prive cloud
+-------------------+
```

**Livrables mi-parcours** :
- Schema architectural
- Repartition detaillee des couts
- Analyse performances (debit, latence, precision)
- Matrice risques (3+ risques avec mitigation)
- Presentation 5 slides defense architecturale

**Budget cloud** : max 45 000 EUR/an (estimations par environnement)

### Phase 4 — Production & Optimisation (5 semaines)
- Deploiement production
- Formation utilisateurs
- Monitoring continu

---

## 9. Budget & Chiffres Cles

| Metrique | Valeur |
|----------|--------|
| Reduction temps | **-70%** |
| Precision cible | **> 92%** |
| Deadline totale | **17 semaines** |
| Budget total | **162k EUR** |
| Budget cloud/an | **max 45k EUR** |

---

## 10. Risques & Mitigation

| Risque | Severite | Mitigation |
|--------|----------|------------|
| Performance insuffisante gros volumes | ELEVE | Tests charge des POC, archi scalable early |
| Taux hallucinations trop eleve | CRITIQUE | RAG strict, validation croisee agents, revue humaine > 80% confiance |
| Complexite orchestration multi-agents | MOYEN | Orchestration simple, tests unitaires, monitoring detaille |
| Depassement budget cloud | MOYEN | Quotas stricts, monitoring couts, option on-premise backup |
| Resistance changement utilisateurs | MOYEN | Implication TEKNO-SAFETY des POC, formation continue, quick wins |

---

## 11. Stack Technique Envisagee

| Composant | Technologie |
|-----------|-------------|
| Embeddings | BERT / Sentence-BERT |
| Base vectorielle | FAISS (POC) → self-hosted Vector DB (prod) |
| LLM | Azure OpenAI (on-tenant) |
| Backend API | FastAPI (Python) |
| Orchestration | Custom multi-agent framework |
| Infrastructure | Azure (tenant prive, VNet, App Container) |
| IaC | Terraform |
| CI/CD | Azure DevOps |
| Parsing PDF | pdfplumber / PyMuPDF |
| NER | spaCy / custom models |
| Monitoring | Azure Monitor |
| Securite | Azure Key Vault, AES-256, NSG |

---

## 12. Equipe

- **Participants Meeting #1** : Sean, Rachelle, Shareez, Lyderic (Equipe Tekno-Family + Client)
- **Lead Data Science** : Sprint 1
- **ML Engineer** : Sprint 2
- **Full Team** : Sprint 3+

---

## 13. Prochaines Etapes Immediates

1. Validation plan + budget par TEKNO-SAFETY
2. Constitution equipe
3. Contrat Tekno-Launch
4. Setup Azure prive
5. Onboarding stagiaire
6. Kick-off avec TEKNO-SAFETY
7. Selection corpus test definitif
