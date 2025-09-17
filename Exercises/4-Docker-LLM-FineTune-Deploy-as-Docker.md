# Exercise: Fine-tuning and Deploying an LLM in Docker fine tuned for Course Name Extraction

## Objective
To fine-tune a pre-trained language model to accurately extract the names of courses offered at a college and deploy it as a Docker container using FastAPI.

## Prerequisites
- Basic understanding of Python, machine learning concepts, and Docker.
- Familiarity with libraries like Hugging Face Transformers and FastAPI.

## Step 1: Set Up the Environment
1. **Install Required Libraries**
   Ensure you have the necessary libraries installed:
   ```bash
   pip install transformers torch datasets fastapi uvicorn
   ```

2. **Import Libraries**
   Start your Python script or Jupyter Notebook by importing the required libraries:
   ```python
   import pandas as pd
   from transformers import AutoModelForTokenClassification, AutoTokenizer
   from fastapi import FastAPI
   from pydantic import BaseModel
   import uvicorn
   ```

## Step 2: Prepare the Dataset
1. **Create a Dataset**
   Create a dataset containing text samples and the corresponding labels indicating course names:
   ```python
   data = {
       "text": [
           "Introduction to Computer Science",
           "Advanced Mathematics",
           "Data Structures and Algorithms",
           "Operating Systems",
           "Machine Learning"
       ],
       "labels": [
           ["Introduction to Computer Science"],
           ["Advanced Mathematics"],
           ["Data Structures and Algorithms"],
           ["Operating Systems"],
           ["Machine Learning"]
       ]
   }

   df = pd.DataFrame(data)
   ```

2. **Convert to Hugging Face Dataset**
   Convert the DataFrame into a Hugging Face dataset:
   ```python
   from datasets import Dataset
   dataset = Dataset.from_pandas(df)
   ```

## Step 3: Tokenization and Fine-tuning
1. **Load a Pre-trained Model and Tokenizer**
   Select a suitable pre-trained model for token classification:
   ```python
   model_name = "dbmdz/bert-large-cased-finetuned-conll03-english"
   tokenizer = AutoTokenizer.from_pretrained(model_name)
   model = AutoModelForTokenClassification.from_pretrained(model_name)
   ```

2. **Tokenize the Dataset**
   Tokenize the dataset:
   ```python
   def tokenize_function(examples):
       return tokenizer(examples["text"], padding="max_length", truncation=True)

   tokenized_dataset = dataset.map(tokenize_function, batched=True)
   ```

3. **Fine-tune the Model**
   Set training arguments and fine-tune the model:
   ```python
   from transformers import Trainer, TrainingArguments

   training_args = TrainingArguments(
       output_dir="./results",
       evaluation_strategy="epoch",
       learning_rate=2e-5,
       per_device_train_batch_size=4,
       num_train_epochs=3,
   )

   trainer = Trainer(
       model=model,
       args=training_args,
       train_dataset=tokenized_dataset,
   )
   trainer.train()
   ```

## Step 4: Create a FastAPI Application
1. **Define the FastAPI Application**
   Create a FastAPI app that exposes an endpoint for course name extraction:
   ```python
   app = FastAPI()

   class CourseRequest(BaseModel):
       text: str

   @app.post("/extract-course-name/")
   async def extract_course_name(request: CourseRequest):
       inputs = tokenizer(request.text, return_tensors="pt", padding=True, truncation=True)
       outputs = model(**inputs)
       predictions = outputs.logits.argmax(dim=-1)
       extracted_names = [request.text]  # Replace with logic to extract names
       return {"extracted_course_names": extracted_names}
   ```

2. **Run the FastAPI App**
   You can run the FastAPI app locally with:
   ```bash
   uvicorn your_script_name:app --reload
   ```

## Step 5: Create a Dockerfile
1. **Create a Dockerfile**
   In the same directory as your FastAPI app, create a file named `Dockerfile` with the following content:
   ```Dockerfile
   FROM python:3.8-slim

   WORKDIR /app

   COPY requirements.txt requirements.txt
   RUN pip install --no-cache-dir -r requirements.txt

   COPY . .

   CMD ["uvicorn", "your_script_name:app", "--host", "0.0.0.0", "--port", "80"]
   ```

2. **Create requirements.txt**
   Create a `requirements.txt` file containing the dependencies:
   ```text
   fastapi
   uvicorn
   transformers
   torch
   datasets
   ```

## Step 6: Build and Run the Docker Container
1. **Build the Docker Image**
   Navigate to the directory containing the Dockerfile and run:
   ```bash
   docker build -t course-extraction-api .
   ```

2. **Run the Docker Container**
   After the image is built, run the container:
   ```bash
   docker run -d -p 80:80 course-extraction-api
   ```

## Step 7: Test the API
1. **Send a POST Request**
   You can test the API using tools like Postman or curl. Here’s an example using curl:
   ```bash
   curl -X POST "http://localhost/extract-course-name/" -H "Content-Type: application/json" -d '{"text": "Introduction to Artificial Intelligence"}'
   ```

## Conclusion
In this exercise, you have learned how to fine-tune a language model for extracting course names and deploy it as a Docker container using FastAPI. You can extend the model’s capabilities by refining the dataset and improving the extraction logic in the FastAPI application. This exercise serves as a practical introduction to working with LLMs and deploying them in real-world applications.

# Q&A based on the exercise

**Q1: What is the primary objective of this exercise?**  
**A1:** The primary objective is to fine-tune a pre-trained language model to accurately extract the names of courses offered at a college and deploy it as a Docker container using FastAPI.

**Q2: What libraries do we need to install to complete this exercise?**  
**A2:** The required libraries include `transformers`, `torch`, `datasets`, `fastapi`, and `uvicorn`. 

**Q3: What type of data do we use for fine-tuning the model?**  
**A3:** We use a dataset that contains text samples of course names and corresponding labels indicating the course names.

**Q4: How do we prepare the dataset for fine-tuning?**  
**A4:** We create a pandas DataFrame with text samples and labels, then convert it into a Hugging Face dataset using `Dataset.from_pandas()`.

**Q5: What function is used to tokenize the dataset?**  
**A5:** The function `tokenize_function()` is used to tokenize the dataset by applying the tokenizer to each text sample, with padding and truncation.

**Q6: Which pre-trained model is selected for this exercise?**  
**A6:** The pre-trained model selected for token classification is `dbmdz/bert-large-cased-finetuned-conll03-english`.

**Q7: What is the role of the FastAPI application in this exercise?**  
**A7:** The FastAPI application serves as a web API that exposes an endpoint for course name extraction. It takes text input and returns the extracted course names.

**Q8: How do we create the Docker container for our FastAPI application?**  
**A8:** We create a Dockerfile that defines the base image, installs the required libraries, and specifies the command to run the FastAPI application. We then build the Docker image and run the container.

**Q9: What command do we use to build the Docker image?**  
**A9:** The command used to build the Docker image is:
```bash
docker build -t course-extraction-api .

