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

This function:
- Moves data and the model to the chosen device (CPU or GPU)
- Computes loss using cross-entropy (appropriate for multi-class classification)
- Tracks accuracy and macro-averaged F1 score using torchmetrics
- Separates training and validation phases

Importantly, none of this code depends on Django. This separation ensures that model training and web serving remain loosely coupled, which is a best practice in machine learning systems.

## Part 3: Saving and loading model weights
Once training is complete, the model’s learned parameters must be saved. Django will later load these parameters to perform inference. A typical way to save weights in PyTorch looks like this:

```python
torch.save(
    {"model_state_dict": model.state_dict()},
    "resnet18_model.pth"
)
```

When serving the model, the architecture must be reconstructed exactly as it was during training. Only then can the saved weights be loaded correctly. This approach is widely recommended because it is more explicit and less error-prone across environments.

## Part 4: Creating a Django view for inference

Now we connect everything together. Django will receive input data, load the trained model, run inference, and return predictions. Below is a Django view that accepts uploaded images and returns predicted labels as JSON:
```python
def analysisPageView(request):
    if request.method == "POST":
        os.makedirs(TEMP_DIR, exist_ok=True)

        for p in TEMP_DIR.iterdir():
            if not p.is_dir():
                continue
            shutil.rmtree(p)

        req_dir = TEMP_DIR / uuid.uuid4().hex
        req_dir.mkdir(parents=True, exist_ok=True)
        
        uploaded_files = request.FILES.getlist('images')

        for f in uploaded_files:
            file_name = Path(f.name).name
            ext = Path(file_name).suffix.lower()
            final_file_name = f'image-{uuid.uuid4().hex[:8]}{ext}'

            with open(req_dir / final_file_name, "wb") as dest:
                for chunk in f.chunks():
                    dest.write(chunk)

        images = load_images_from_path(req_dir)
        test_tfms = get_default_test_transforms()
        device = "cuda"

        model = ResNet18_CustomHead(num_classes=5).to(device)

        ckpt = torch.load(
            MODEL_WEIGHTS_PATH / "resnet18_model.pth",
            map_location=device
        )
        model.load_state_dict(ckpt["model_state_dict"])
        model.eval()

        pred_labels = infer_on_unknown_data(
            images, model, device, test_tfms
        )

        return JsonResponse({"labels": pred_labels})

    return render(request, "app/analysis.html")
```

Note again:
- Loading the model inside the request handler is acceptable for demonstrations but inefficient for high-traffic systems.
- `model.eval()` is required to disable training-specific layers such as dropout.
- Preprocessing must exactly match what was used during training, or predictions may be unreliable.

## Part 5: Testing the API
Once the view is wired into urls.py, you can test it by:
- Submitting images via an HTML form
- Sending a POST request using Postman or curl
- Calling the endpoint from a frontend application

If everything is set up correctly, Django will return predictions as structured JSON, making it easy to integrate with other services.

Here is an example view to use model inference:
```python
def ModelInferenceView():
  ...
  images = load_images_from_path(req_dir)

  ## Warning: not recommended for production
  model = ResNet18_CustomHead(
    num_classes=5
  ).to(device)

  ckpt = torch.load(MODEL_WEIGHTS_PATH / "resnet18_model.pth", map_location=device)
  state_dict = ckpt["model_state_dict"]
  model.load_state_dict(state_dict)
  ...
  pred_labels = infer_on_unknown_data(images, model, device, test_tfms)

  return JsonResponse({'labels': pred_labels})
```

## Recap
In this article, we walked through the full lifecycle of serving a deep learning model using Django:
- Created a Django project and application
- Trained a PyTorch model outside the web layer
- Saved and loaded model weights correctly
- Exposed a Django view for inference
- Tested the model via HTTP requests

Django is not designed for large-scale model serving, but it is a practical choice for prototypes, internal tools, and workloads where simplicity and flexibility matter.

That's it for today. The reference code (related, but for different project) is available [here](https://github.com/oncs21/remote-weather-app "Remote Weather App"). If you found this useful, consider following me on [LinkedIn](https://www.linkedin.com/in/onapte/) and starring (⭐) the repository.
