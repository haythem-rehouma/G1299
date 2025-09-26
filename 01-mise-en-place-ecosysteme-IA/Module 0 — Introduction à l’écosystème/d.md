
# Introduction à Linux pour l'Intelligence Artificielle

Apprenez à utiliser Linux pour **développer**, **entraîner** et **déployer** des projets d’IA.  
Ce guide est pensé pour être **pratique**, même si vous n’êtes pas expert en Linux.

---

## Ressources complémentaires

Les exercices, scripts d’installation et configurations détaillées sont disponibles dans le repository associé à ce cours.  
👉 Pas besoin de tout taper à la main : vous pouvez réutiliser directement les fichiers fournis.

---

## Table des matières

1. Pourquoi utiliser Linux pour l’IA ?
2. Les outils d’IA disponibles sous Linux
3. Quelles distributions choisir pour démarrer ?
4. Gérer ses environnements Python
5. Utiliser le GPU pour aller plus vite
6. Isoler ses projets avec Docker ou Kubernetes
7. Les bibliothèques indispensables
8. Organiser et stocker ses données
9. Déployer un modèle en production
10. Surveiller et optimiser ses entraînements
11. Exemples pratiques et scripts prêts à l’emploi

---

## 1. Pourquoi utiliser Linux pour l’IA ?

Linux est **le système préféré des chercheurs et ingénieurs IA**.  
Pourquoi ? Parce qu’il est :  

- **Rapide et optimisé** : il gère bien le CPU et le GPU.  
- **Ouvert et gratuit** : tout est open source, donc pas de licences coûteuses.  
- **Flexible** : il fonctionne aussi bien sur un PC portable que sur un cluster géant.  
- **Reproductible** : la même configuration peut être utilisée partout (local, cloud, HPC).  

💡 Exemple : si vous entraînez un modèle PyTorch sur votre machine, vous pouvez facilement réutiliser le même environnement sur un serveur cloud.

---

## 2. Les outils d’IA disponibles sous Linux

Voici les grands outils que vous croiserez :  

- **Applications** : Jupyter (notebooks), MLflow (suivi d’expériences), Streamlit et Gradio (interfaces simples pour vos modèles).  
- **Frameworks IA** : PyTorch, TensorFlow, scikit-learn, XGBoost.  
- **Calcul parallèle** : CUDA (NVIDIA), ROCm (AMD), Dask ou Ray pour distribuer vos calculs.  
- **Infrastructure** : Docker (containers), Kubernetes (orchestration), SLURM (HPC).  

---

## 3. Quelles distributions choisir pour démarrer ?

- **Ubuntu 22.04 LTS** : la plus simple pour commencer (bonne doc, compatible GPU).  
- **Pop!\_OS** : pratique sur poste de travail, surtout si vous avez une carte graphique NVIDIA.  
- **Rocky / AlmaLinux** : pour les serveurs (stables, compatibles RHEL).  
- **Lambda Stack** : pensée pour l’IA, avec PyTorch/TensorFlow déjà installés.  

👉 Conseil : commencez avec **Ubuntu**, c’est la plus utilisée.

---

## 4. Gérer ses environnements Python

Le problème classique : *“ça marchait hier, mais plus aujourd’hui…”* → conflits de versions.  
Pour éviter ça :  

- **Conda/Mamba** : parfait pour tester et explorer (un environnement par projet).  
- **Poetry** : mieux si vous développez une librairie partagée.  
- **Docker + venv** : idéal pour la production (images stables et portables).  

---

## 5. Utiliser le GPU pour aller plus vite

En IA, le GPU est **indispensable** pour entraîner rapidement vos modèles.  

- **NVIDIA (CUDA)** : le plus courant et le plus mature (ex. cuDNN, TensorRT).  
- **AMD (ROCm)** : une alternative ouverte, mais dépend du matériel.  
- **Multi-GPU** : si vous avez plusieurs cartes, vous pouvez les faire travailler ensemble avec NCCL, Gloo ou MPI.  

💡 Testez si PyTorch voit votre GPU :  
```python
import torch
print(torch.cuda.is_available())
````

---

## 6. Isoler ses projets avec Docker ou Kubernetes

* **Docker** : crée des environnements reproductibles → *“ça marche chez moi” devient “ça marche partout”*.
* **Kubernetes** : utile quand on a beaucoup de projets à déployer ou des jobs d’entraînement à distribuer.
* **Singularity** : recommandé dans les clusters HPC où on n’a pas les droits administrateurs.

---

## 7. Les bibliothèques indispensables

* **Machine Learning** : scikit-learn, XGBoost, LightGBM.
* **Deep Learning** : PyTorch (souvent préféré), TensorFlow/Keras.
* **Visualisation** : Matplotlib, Seaborn, Plotly.
* **Suivi d’expériences** : MLflow, Weights & Biases.

---

## 8. Organiser et stocker ses données

Bonnes pratiques :

* Séparer **brut / traité / résultats**.
* Versionner avec **DVC** ou **Git LFS**.
* Utiliser des formats efficaces comme **Parquet**.
* Stocker les artefacts (modèles, checkpoints) dans **S3, GCS ou MinIO**.

---

## 9. Déployer un modèle en production

* **Serving** : FastAPI, TorchServe, TensorFlow Serving, Triton.
* **Automatisation (CI/CD)** : GitHub Actions ou GitLab CI pour tester, builder et déployer automatiquement.
* **Surveillance** : Prometheus + Grafana pour les métriques et alertes.

---

## 10. Surveiller et optimiser ses entraînements

* **Sur la machine** : `htop` (CPU/RAM), `nvidia-smi` (GPU).
* **Pour les modèles** : profiler PyTorch/TensorFlow, quantification, pruning, ONNX.
* **Optimisation GPU** : TensorRT pour accélérer l’inférence.

---

## 11. Exemples pratiques et scripts prêts à l’emploi

Le repo contient des scripts déjà prêts pour tester :

* `scripts/env_cpu.sh` → créer un environnement Python minimal (CPU).
* `scripts/pytorch_cuda.sh` → installer PyTorch + CUDA et tester le GPU.
* `scripts/mlflow_local.sh` → lancer MLflow en local.
* `docker/Dockerfile` → exemple d’image Docker pour servir un modèle.

👉 Vous pouvez directement exécuter ces fichiers pour gagner du temps.

---

```



