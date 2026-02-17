# 🥔 Rotten Potatoes: MLOps-Powered Movie Review Hub

**Rotten Potatoes** is a full-stack application that enables users to browse movies and receive real-time sentiment scores on their reviews. Beyond the UI, it serves as a comprehensive implementation of a modern **MLOps pipeline**, automating the lifecycle from data versioning to production deployment on Kubernetes.

The core objective of this project was to build a reproducible, self-healing, and automated infrastructure rather than just a standalone web app. It orchestrates the entire software lifecycle—data versioning, model training, container building, and deployment—without manual intervention.

## 🛠️ Tech Stack & Key Features

* **Full-Stack Application:** Built with **FastAPI** (Backend) and **Streamlit** (Frontend) backed by **PostgreSQL**.
* **CI/CD Pipeline:** Fully automated workflow orchestrated by **Jenkins** and configured via **Ansible** playbooks.
* **MLOps & Data Versioning:**
    * **DVC (Data Version Control)** ensures training data immutability and reproducibility.
    * **MLflow** tracks experiments and manages model artifact promotion.
* **Infrastructure:**
    * **Docker & Kubernetes (Minikube):** Containerized microservices with Namespace isolation.
    * **Autoscaling:** Implements Horizontal Pod Autoscaling (HPA) to handle traffic spikes.
* **Observability:** Integrated **ELK Stack (Elasticsearch & Kibana)** for structured logging and real-time system monitoring.
  
Kindly refer ```SPE_Project_Report.pdf``` for more details.
***Project Structure:***
```
.
├── ansible
│   ├── inventory.ini
│   ├── playbook.yml
│   ├── roles
│   │   ├── build_and_push_to_docker
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── configure_kibana
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── deploy_on_kubernetes
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── install_docker
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── install_kubernetes
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── prepare_kubernetes
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── set_up_workspace
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── train_model
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── update_system
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   └── use_latest_model
│   │       └── tasks
│   │           └── main.yml
│   └── rotpot_vault.yml
├── app.py
├── data
│   ├── full_dataset.csv
│   ├── IMDB Dataset.csv.zip
│   ├── initial_movies.json
│   ├── initial_reviews.json
│   ├── train.csv
│   └── train.csv.dvc
├── docker
│   ├── docker-compose.yml
│   ├── Dockerfile.backend
│   └── Dockerfile.frontend
├── frontend.py
├── Jenkinsfile
├── kubernetes
│   ├── k8s-database.yaml
│   ├── k8s-ingress.yaml
│   ├── k8s-logging.yaml
│   └── templates
│       ├── backend.yaml.j2
│       └── frontend.yaml.j2
├── manage_data.py
├── notes.txt
├── problem_statement.pdf
├── report.pdf
├── requirements.txt
└── train.py
```
