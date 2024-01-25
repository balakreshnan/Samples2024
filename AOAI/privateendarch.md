# Generation AI using Azure Open AI Secured Deployment Architecture

## Introduction

- Build a end to end architecture
- Using RAG Retrival Augmented Generation pattern for chat bot
- Using Azure Open AI services (Any Model)
- Using Azure AI search for vector index search
- Using Cosmos DB for Storing Meta data and other information

## Architecture

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/aoaisolprivateendpoint.jpg 'RagChat')

## Details

- Understanding Traffic flow
- Inbound traffic flow needs private endpoint
- Outbound is vnet integration
- One VNET with 2 subnets one for all private endpoint for inbound traffic and another for outbound traffic
- At least 2 subnets /28
- Enabling Routing
- VNET peering to hub VNET
- Network security groups are in place
- 1 subnet for all resource, open, ai search, cosmos, web app
- Private DNS zone also created at the same time
- Or associate create private endpoint with DNS private zone
- VNET Integration for outbound traffic for APP service or web application
- DNS is most of the work
- Azure DNS server for private endpoint resolvers
- Within the VNET default is azure DNS 
- Within the VNET to private DNS zone
- Every one should be able to resolve the DNS in that private VNET
- On premise configure conditional forwarders - pointing to Private dns resolvers in Azure
- Inbound endpoint to private DNS resolver from their it will resolve to azure private dns solvers