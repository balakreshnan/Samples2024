# Gen AI Application vector Management

## Introduction

- Strategy to manage vectors in AI applications
- Manage multiple vectors based on domain information
- Create and Manage embedding
- Manage embeddings lifecycle
- Security in AI vector database or store
- Model based vector management
- Multi Model management
- Linege of how the vectors are used in applications
- Other considerations

## Strategy to manage vectors in AI applications

- For Rag based gen ai applications we need to able to take pdf, word, or other documents and convert them to vectors
- Text vectors are created based on open ai ada 002 model or new models
- Image vectors are created based on open ai clip or azure ai vision florence model v4.0
- Audio vectors are created based on open ai jukebox or azure ai speech model
- Define a process to create embeddings.
- Creating large document embeddings needs some attention to manage api throttling
- We can leverage few accelerators to create embeddings for large volume documents
- Above also applies to image and audio embeddings
- Embeddings will depend on what model is used in open ai or other LLM models
- Have a documented process to maintain meta data for vectors like vector name, business domain or application domain it's for.
- Security on vectors is still challengin but there are way to have separarte columns to store security predicates.
- Azure AI search has security predicates to manage security on vectors using propeties columns
- also have a system to document which vectors are used in which applications
- Have a frequent cycle to update vectors into a new embedding models based on newer LLM models
- Or some times vendor might say they are retiring older models.
- as of writing this article text, image and audio vectors are different and managed differently
- Create a automated process to convert the existing vectors one by one in a index
- Test the application using the new vectors
- Have a rollback plan if the new vectors are not working as expected
- Have a process to manage the lineage of vectors in the application
- Now if successful then do a migration plan to move all the vectors to the new model
- Plan the downtime as minimal as possible
- Categorize vectors based on business domain or application domain
- Vecor database or index API version management with new features and it'e lifecycle