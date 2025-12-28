---
title: "Serving a deep learning model with Django"
date: 2025-12-28
---

Deep learning did not suddenly appear with large language models. The field has been evolving for decades, starting with early neural network research in the late 20th century and gradually improving as compute, data, and algorithms advanced. Today, deep learning systems are used in image recognition, natural language processing, recommendation systems, and many other real-world applications. Training a model, however, is only part of the workflow. To make a model useful, it must be served. It needs to accept inputs and return predictions in a reliable way. In this article, we will walk through a practical approach to serving a deep learning model using Django and PyTorch.

Note: The focus here is clarity and correctness rather than production-scale optimization (we will see this in a future article).

## Part 1: Set up a Django project
We start by creating a standard Django project. Django works well for this use case because it already provides request handling, routing, and serialization tools. Create and activate a virtual environment, then install Django:

```bash
python -m venv venv
source venv/bin/activate
pip install django
```

Next, create a new Django project:
```bash
django-admin startproject sample_project
cd sample_project
```

Inside the project, create an application that will contain our model-serving logic:
```bash
python manage.py startapp my_app
```

For this project, our directory should look like this:
```
sample_project/
├── sample_project/
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── my_app/
│   ├── views.py
│   ├── apps.py
│   └── templates/
├── manage.py
```

Finally, add my_app to the **INSTALLED_APPS** list in `settings.py`. At this stage, no deep learning code is involved yet.

## Part 2: Train a deep learning model
Model training is intentionally kept outside Django. Training is usually compute-heavy and better handled in a separate script or notebook. Django’s responsibility will later be inference, not optimization. For this example, we use a PyTorch model. You may train your own model or adapt a pre-trained one depending on your task. Below is a training function that performs supervised learning with validation and tracks common metrics.

For now, we will focus on using a PyTorch model. You can either use a trained or a pre-trained model.

```python
def train_model(model: nn.Module,
                loss_fn,
                optim,
                lr: float,
                train_dl,
                val_dl,
                num_classes: int,
                epochs: int = 20,
                device: str = "cuda",
                ) -> dict[str, list[float]]:

```
