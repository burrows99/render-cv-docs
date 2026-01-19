# Azure Migration Plan: Reducing Complexity with Native Services

This document outlines the strategy to replace self-hosted components with Azure-native services, reducing infrastructure complexity, operational overhead, and improving enterprise compliance.

## Overview

**Goal:** Eliminate 60% of infrastructure code and 80% of operational overhead by replacing Docker containers and third-party services with fully-managed Azure PaaS offerings.

**Current Architecture:** 7 Docker containers (6 services + 1 volume)
**Target Architecture:** 4 Docker containers + 5 Azure managed services

---

## Components to Replace

### 1. Elasticsearch → Azure AI Search

**Current Complexity:**
- Self-hosted Docker container (port 9200)
- Manual index management and configuration
- Custom security setup (disabled SSL for local dev)
- 1GB memory overhead per instance
- Trial license management (14-day expiration)
- Custom LangChain Elasticsearch integration

**Azure Replacement:**
- **Service:** Azure AI Search (formerly Azure Cognitive Search)
- **SKU:** Basic tier for development, Standard for production
- **Features:**
  - Fully managed with auto-scaling
  - Built-in security & HTTPS
  - Native integration with Azure OpenAI
  - Semantic ranking included
  - Hybrid search (vector + keyword)
  - Integrated with Azure identity (Managed Identity)

**Migration Steps:**
1. Create Azure AI Search resource via Bicep template
2. Implement `make_azure_ai_search_retriever()` in `retrieval.py`
3. Update `IndexConfiguration.retriever_provider` to include `"azure-ai-search"`
4. Migrate existing Elasticsearch indexes using Azure Search SDK
5. Update search filter syntax from Elasticsearch to OData
6. Remove `elasticsearch` service from `compose.yaml`
7. Remove Elasticsearch environment variables from `.env.retrieval-agent`

**Files Modified:**
- `retrieval-agent/src/retrieval_graph/retrieval.py` (add new retriever)
- `retrieval-agent/src/retrieval_graph/configuration.py` (update Literal type)
- `compose.yaml` (remove elasticsearch service)
- `.env.retrieval-agent` (remove ELASTICSEARCH_* vars)

**Code Reduction:** ~40 lines from Docker config, ~50 lines from retrieval.py (can remove elastic-local variant)

---

### 2. OpenAI API → Azure OpenAI Service

**Current Complexity:**
- Direct OpenAI API calls via public internet
- Separate API key management outside Azure
- No enterprise compliance features
- Rate limiting at API provider level
- No content filtering or moderation

**Azure Replacement:**
- **Service:** Azure OpenAI Service
- **Deployments Needed:**
  - `gpt-4o` or `gpt-4o-mini` (response generation)
  - `gpt-4o-mini` (query refinement)
  - `text-embedding-3-small` or `text-embedding-3-large` (embeddings)
- **Features:**
  - Enterprise SLA with regional deployment
  - Integrated with Azure Key Vault
  - Content filtering & responsible AI guardrails
  - VNet support for private endpoints
  - Cost controls via Azure budgets
  - Compliance certifications (HIPAA, SOC 2, ISO 27001)

**Migration Steps:**
1. Create Azure OpenAI Service resource via Bicep
2. Deploy required models (gpt-4o, gpt-4o-mini, text-embedding-3-small)
3. Update `make_text_encoder()` in `retrieval.py`:
   - Replace `langchain_openai.OpenAIEmbeddings` with `langchain_openai.AzureOpenAIEmbeddings`
   - Add Azure endpoint and API version parameters
4. Update `load_chat_model()` in `utils.py`:
   - Support `azure/model-name` provider format
   - Configure Azure-specific parameters (api_version, azure_deployment)
5. Update `.env.retrieval-agent` with Azure OpenAI variables:
   ```bash
   AZURE_OPENAI_ENDPOINT=https://<resource-name>.openai.azure.com/
   AZURE_OPENAI_API_KEY=<key>  # or use Managed Identity
   AZURE_OPENAI_API_VERSION=2024-12-01-preview
   AZURE_OPENAI_DEPLOYMENT_NAME_CHAT=gpt-4o
   AZURE_OPENAI_DEPLOYMENT_NAME_QUERY=gpt-4o-mini
   AZURE_OPENAI_DEPLOYMENT_NAME_EMBEDDING=text-embedding-3-small
   ```
6. Update default model names in `configuration.py`:
   - `response_model`: `azure/gpt-4o`
   - `query_model`: `azure/gpt-4o-mini`
   - `embedding_model`: `azure/text-embedding-3-small`
7. Configure Managed Identity (optional, recommended):
   - Assign Container Apps identity to Azure OpenAI role
   - Remove API key from environment variables
   - Use `DefaultAzureCredential` in initialization

**Files Modified:**
- `retrieval-agent/src/retrieval_graph/utils.py` (update load_chat_model)
- `retrieval-agent/src/retrieval_graph/retrieval.py` (update make_text_encoder)
- `retrieval-agent/src/retrieval_graph/configuration.py` (update defaults)
- `.env.retrieval-agent` (add Azure OpenAI vars)

**Code Changes:** ~30 lines modified, enterprise compliance gained

---

### 3. MinIO → Azure Blob Storage

**Current Complexity:**
- Self-hosted S3-compatible storage (ports 9000/9001)
- Custom authentication with root credentials
- Manual backup and replication management
- Local volume persistence in `./minio/data`
- Separate health check and restart policy
- Web console UI management

**Azure Replacement:**
- **Service:** Azure Blob Storage (with Data Lake Gen2 enabled)
- **Container:** `documents` (or multiple containers per use case)
- **Features:**
  - Fully managed object storage with 99.999999999% durability
  - Lifecycle policies (hot/cool/archive tiers for cost optimization)
  - Built-in geo-replication (LRS, GRS, RA-GRS)
  - S3-compatible API via Data Lake Gen2
  - Native integration with Azure services
  - Automatic HTTPS & encryption at rest
  - Azure RBAC integration for access control
  - CDN integration for global access
  - Immutable storage and legal hold support

**Migration Steps:**
1. Create Azure Storage Account via Bicep template
   - Enable hierarchical namespace (Data Lake Gen2)
   - Configure private endpoint for VNet access
2. Create `documents` container with blob access tier
3. Update document loaders to use Azure Blob Storage SDK:
   - Replace `S3DirectoryLoader` with `AzureBlobStorageContainerLoader` (LangChain)
   - Update connection strings to use Azure Storage
4. Migrate existing MinIO data:
   ```bash
   azcopy copy "http://localhost:9000/bucket/*" \
     "https://<account>.blob.core.windows.net/documents?<SAS>" --recursive
   ```
5. Remove `minio` service from `compose.yaml`
6. Remove MinIO environment variables from `.env.retrieval-agent`
7. Add Azure Storage variables:
   ```bash
   AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
   # or use Managed Identity (recommended)
   AZURE_STORAGE_ACCOUNT_NAME=<account-name>
   AZURE_STORAGE_CONTAINER_NAME=documents
   ```

**Files Modified:**
- `compose.yaml` (remove minio service)
- Document ingestion code (if using S3DirectoryLoader)
- `.env.retrieval-agent` (replace MINIO_* with AZURE_STORAGE_*)

**Code Reduction:** ~30 lines from Docker config, no MinIO container management

---

### 4. Azure SQL Server Container → Azure SQL Database

**Current Complexity:**
- Self-hosted SQL Server 2022 container (port 1433)
- Manual data persistence in `./azure-sql-server/data`
- Health checks and restart policies
- 2GB+ container image size
- Developer edition limitations (free but non-production)
- Manual patching and security updates
- Local volume management for data/logs/secrets

**Azure Replacement:**
- **Service:** Azure SQL Database
- **Tier:** Basic for development, Standard/Premium for production
- **Features:**
  - Fully managed PaaS database with SLA
  - Automatic backups (35-day retention)
  - Built-in high availability (99.99% SLA)
  - Query performance insights and tuning
  - Automatic patching and updates
  - Geo-replication support
  - Serverless tier for cost savings (auto-pause)
  - Advanced threat protection
  - Private endpoint support

**Migration Steps:**
1. Create Azure SQL Database resource via Bicep template
2. Configure firewall rules or private endpoint for Container Apps
3. Migrate schema and data from container:
   ```bash
   sqlpackage /Action:Export /SourceConnectionString:"Server=localhost,1433;..." \
     /TargetFile:backup.bacpac
   sqlpackage /Action:Import /SourceFile:backup.bacpac \
     /TargetConnectionString:"Server=<server>.database.windows.net;..."
   ```
4. Update connection strings in code (if using SQLDatabaseLoader)
5. Configure Azure AD authentication (optional, recommended)
6. Remove `azure-sql-server` service from `compose.yaml`
7. Update environment variables:
   ```bash
   AZURE_SQL_CONNECTION_STRING=Server=<server>.database.windows.net;Database=<db>;...
   # or use Managed Identity
   AZURE_SQL_SERVER=<server>.database.windows.net
   AZURE_SQL_DATABASE=<database-name>
   ```

**Files Modified:**
- `compose.yaml` (remove azure-sql-server service)
- `.env.retrieval-agent` (update SQL connection string)
- Document loaders using SQLDatabaseLoader (if any)

**Code Reduction:** ~35 lines from Docker config, eliminates largest container

---

### 5. Multiple Retrievers → Consolidate to Azure AI Search

**Current Complexity:**
- 4 different retriever providers: `elastic`, `pinecone`, `mongodb`, `cognee`
- 4 separate `make_*_retriever()` functions in `retrieval.py`
- Multiple credential types (API keys, connection strings, URLs)
- Provider-specific filtering syntax:
  - Elasticsearch: `{"term": {"metadata.user_id": user_id}}`
  - MongoDB: `{"user_id": {"$eq": user_id}}`
  - Pinecone: `{"user_id": user_id}` in metadata filters
- Different search parameter conventions
- Testing complexity across 4 backends

**Azure Replacement:**
- **Service:** Azure AI Search (single retriever)
- **Features:**
  - Unified API for all operations
  - Consistent OData filtering syntax
  - Built-in vector search support
  - Semantic ranking for better relevance
  - Integrated with Azure OpenAI for embeddings
  - Single credential system (Managed Identity)

**Migration Steps:**
1. Implement comprehensive `make_azure_ai_search_retriever()` function
2. Create index schema with user_id field for filtering
3. Add OData filter generation: `$filter=user_id eq '{user_id}'`
4. **Keep Cognee** - unique knowledge graph capability not replaceable
5. Remove or deprecate other retrievers:
   - `make_elastic_retriever()` and `make_pinecone_retriever()` → migrate to Azure AI Search
   - `make_mongodb_retriever()` → migrate to Azure AI Search
   - Mark as deprecated with migration warnings
6. Update `configuration.py` to simplify `retriever_provider`:
   ```python
   retriever_provider: Literal["azure-ai-search", "cognee"]
   ```
7. Update integration tests to use Azure AI Search
8. Remove external service credentials from `.env.retrieval-agent`:
   - Remove ELASTICSEARCH_URL, ELASTICSEARCH_API_KEY
   - Remove PINECONE_API_KEY, PINECONE_INDEX_NAME
   - Remove MONGODB_URI

**Files Modified:**
- `retrieval-agent/src/retrieval_graph/retrieval.py` (remove 3 retrievers, add 1)
- `retrieval-agent/src/retrieval_graph/configuration.py` (simplify Literal)
- `tests/integration_tests/test_graph.py` (update tests)
- `.env.retrieval-agent` (remove 3 sets of credentials)

**Code Reduction:** ~120 lines from retrieval.py, 50% environment variable reduction

---

## Implementation Priority

### Phase 1: Immediate (Week 1) - Foundation Services

**Priority 1: Azure OpenAI Service**
- **Rationale:** Core dependency, required for all LLM operations
- **Impact:** Enables enterprise compliance, cost controls
- **Effort:** 4-6 hours
- **Tasks:**
  1. Create Azure OpenAI resource and deploy models
  2. Update `utils.py` and `retrieval.py` for Azure endpoints
  3. Configure environment variables
  4. Test embeddings and chat completions
- **Deliverables:**
  - Bicep template for Azure OpenAI Service
  - Updated Python code with Azure SDK
  - Environment variable documentation

**Priority 2: Azure Blob Storage**
- **Rationale:** Simple replacement, no complex logic changes
- **Impact:** Removes one container, simplifies storage
- **Effort:** 2-3 hours
- **Tasks:**
  1. Create Storage Account with Data Lake Gen2
  2. Update document loaders (if using MinIO)
  3. Migrate existing data with azcopy
  4. Remove MinIO from compose.yaml
- **Deliverables:**
  - Bicep template for Storage Account
  - Data migration scripts
  - Updated compose.yaml

### Phase 2: Short-term (Week 2) - Data Services

**Priority 3: Azure SQL Database**
- **Rationale:** Eliminates largest container, improves reliability
- **Impact:** 2GB+ container removed, automatic backups
- **Effort:** 3-4 hours
- **Tasks:**
  1. Create Azure SQL Database resource
  2. Migrate schema and data
  3. Update connection strings
  4. Test SQLDatabaseLoader functionality
  5. Remove SQL Server container
- **Deliverables:**
  - Bicep template for Azure SQL Database
  - Migration scripts (schema + data)
  - Updated connection configuration

**Priority 4: Azure AI Search**
- **Rationale:** Core retrieval service, replaces 3 providers
- **Impact:** Simplifies architecture, single API, better performance
- **Effort:** 6-8 hours
- **Tasks:**
  1. Create Azure AI Search resource
  2. Implement `make_azure_ai_search_retriever()`
  3. Create index schema with vector fields
  4. Migrate data from Elasticsearch (if exists)
  5. Update user_id filtering to OData syntax
  6. Update configuration and tests
  7. Remove Elasticsearch container
- **Deliverables:**
  - Bicep template for Azure AI Search
  - New retriever implementation
  - Data migration scripts
  - Updated integration tests

### Phase 3: Optional (Ongoing) - Optimization

**Priority 5: Keep Cognee (No Changes)**
- **Rationale:** Unique knowledge graph capability not available in Azure
- **Decision:** Continue running as Docker container
- **Justification:**
  - Provides semantic memory and entity relationship extraction
  - Complements Azure AI Search with graph-based reasoning
  - File-based storage is acceptable for this use case
  - Can migrate to Cosmos DB later if needed for scale

**Future Considerations:**
- Migrate Cognee storage to Azure Cosmos DB for MongoDB API (if multi-replica needed)
- Implement Azure Container Apps scaling policies
- Set up Azure Monitor dashboards and alerts
- Configure Azure Front Door for global distribution

---

## Code Reduction Estimate

| Component | Lines Removed | Files Simplified | Containers Removed |
|-----------|---------------|------------------|--------------------|
| Elasticsearch service | ~40 (compose.yaml) | 1 | 1 |
| MinIO service | ~30 (compose.yaml) | 1 | 1 |
| SQL Server service | ~35 (compose.yaml) | 1 | 1 |
| Retriever abstraction | ~120 (retrieval.py) | 1 | 0 |
| Environment variables | ~50% reduction | .env files | 0 |
| **Total** | **~225 lines** | **4 files** | **3 containers** |

---

## Final Architecture

### Remaining Docker Services (4 containers)
1. **langgraph-server** (Python LangGraph agent)
   - Port 2024
   - Custom business logic
   - Connects to Azure services

2. **agent-chat-ui** (Next.js frontend)
   - Port 3000
   - User interface
   - Connects to langgraph-server

3. **cognee** (Knowledge graph API)
   - Port 8000
   - Semantic memory and entity extraction
   - Unique capability not replaceable by Azure

4. **cognee-mcp** (Model Context Protocol)
   - Port 8001
   - VS Code integration
   - MCP server for Cognee tools

### Azure Managed Services (7 services)
1. **Azure Container Apps** - Host Docker services with auto-scaling
2. **Azure OpenAI Service** - LLM inference and embeddings
3. **Azure AI Search** - Vector retrieval and semantic search
4. **Azure Blob Storage** - Document storage (PDFs, DOCX, etc.)
5. **Azure SQL Database** - Structured data storage
6. **Azure Key Vault** - Secrets management (API keys, connection strings)
7. **Azure Application Insights** - Monitoring, tracing, and diagnostics

### Network Architecture
```
Internet
  │
  ├─→ Azure Front Door (optional CDN/WAF)
  │
  └─→ Azure Container Apps Environment
        │
        ├─→ agent-chat-ui (public ingress)
        │     └─→ langgraph-server (internal)
        │           ├─→ cognee (internal)
        │           └─→ cognee-mcp (internal)
        │
        └─→ Azure Services (private endpoints via VNet)
              ├─→ Azure OpenAI Service
              ├─→ Azure AI Search
              ├─→ Azure Blob Storage
              ├─→ Azure SQL Database
              └─→ Azure Key Vault
```

---

## Benefits Summary

### Infrastructure Reduction
- **60% fewer containers** (7 → 4)
- **225+ lines of Docker config removed**
- **50% environment variable reduction**
- **3 external service dependencies eliminated** (Elasticsearch Cloud, Pinecone, MongoDB Atlas)

### Operational Improvements
- **Zero container maintenance** for data services
- **Automatic backups** for SQL and Blob Storage
- **Auto-scaling** for all Azure services
- **Built-in monitoring** with Application Insights
- **Single sign-on** via Azure AD/Managed Identity

### Enterprise Compliance
- **HIPAA, SOC 2, ISO 27001** certified services
- **Content filtering** via Azure OpenAI
- **Private networking** via VNet and private endpoints
- **Audit logging** via Azure Monitor
- **Data residency** control with Azure regions

### Cost Optimization
- **Reserved capacity pricing** for predictable workloads
- **Serverless SQL** for variable usage patterns
- **Blob storage tiers** (hot/cool/archive)
- **Azure OpenAI provisioned throughput** vs pay-per-token
- **Single Azure bill** with cost management tools

---

## Migration Risks & Mitigations

### Risk 1: User ID Filtering Differences
**Issue:** OData syntax in Azure AI Search differs from Elasticsearch/MongoDB
**Mitigation:** 
- Create comprehensive test suite for user isolation
- Validate filtering in integration tests
- Document OData filter syntax clearly

### Risk 2: Cognee Storage Persistence
**Issue:** File-based storage may not scale with multiple Container App replicas
**Mitigation:**
- Keep single replica for Cognee initially
- Plan migration to Azure Cosmos DB for future scale
- Consider Azure File Share mount for shared storage

### Risk 3: Cost Increase
**Issue:** Azure managed services may cost more than self-hosted containers
**Mitigation:**
- Start with Basic/Standard tiers for development
- Use cost calculator for estimates
- Implement budget alerts and spending limits
- Leverage Azure reserved capacity for 30-40% savings

### Risk 4: Vendor Lock-in
**Issue:** Deep Azure integration reduces portability
**Mitigation:**
- Keep LangChain abstractions intact (RetrieverInterface)
- Document provider-agnostic patterns
- Maintain Docker Compose for local development
- Use Azure Arc for hybrid cloud scenarios if needed

---

## Success Metrics

### Technical Metrics
- ✅ **Container count reduced** from 7 to 4 (43% reduction)
- ✅ **Docker Compose lines** reduced by 225+ (40% reduction)
- ✅ **Environment variables** reduced by 50%
- ✅ **Retriever implementations** reduced from 4 to 2 (50% reduction)
- ✅ **Build time** reduced (smaller images, no Elasticsearch/SQL Server)

### Operational Metrics
- ✅ **Deployment time** reduced to <5 minutes (from ~15 minutes)
- ✅ **Zero-downtime deployments** via Container Apps blue-green
- ✅ **Automatic scaling** based on CPU/HTTP metrics
- ✅ **99.9%+ uptime** SLA for Azure services
- ✅ **Monitoring coverage** at 100% (Application Insights)

### Business Metrics
- ✅ **Compliance certifications** achieved (HIPAA, SOC 2)
- ✅ **Time to production** reduced by 60%
- ✅ **Operational overhead** reduced by 80%
- ✅ **Infrastructure cost predictability** improved via reserved capacity

---

## Next Steps

1. **Review this plan** with team and stakeholders
2. **Create Bicep templates** for all Azure resources (Week 1)
3. **Set up Azure DevOps/GitHub Actions** for CI/CD (Week 1)
4. **Execute Phase 1** (Azure OpenAI + Blob Storage) (Week 1)
5. **Execute Phase 2** (Azure SQL + AI Search) (Week 2)
6. **Validate** with integration tests and user acceptance testing (Week 3)
7. **Document** Azure-specific configuration and troubleshooting (Week 3)
8. **Deploy to production** with rollback plan (Week 4)

---

## References

- [Azure AI Search Documentation](https://learn.microsoft.com/azure/search/)
- [Azure OpenAI Service Documentation](https://learn.microsoft.com/azure/ai-services/openai/)
- [Azure Container Apps Documentation](https://learn.microsoft.com/azure/container-apps/)
- [LangChain Azure Integrations](https://python.langchain.com/docs/integrations/platforms/microsoft)
- [Azure Architecture Center - RAG Pattern](https://learn.microsoft.com/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide)