# Self-hosted Open-WebUI with Ollama on GitHub Codespace
## Conceptual Diagram
```mermaid
flowchart TD
    %% External actors
    subgraph Public Internet

        %% Cloudflare side
        subgraph CloudflareEdge
            cf["Cloudflare Zero Trust"]
        end

        subgraph OpenAI
            openai["LLM API Endpoints"]
        end
    end

    %% End-users
    subgraph User
        user["fa:fa-user Prompt Engineer\n(web browser)"]
    end

    %% Corporate network behind firewall
    subgraph Codespace or DevContainer

        subgraph Docker Bridge Network

            subgraph routing
                firewall["Cloudflared"]
            end

            subgraph reverse-proxy
                caddy["Caddy \n<small> Auto-TLS via Let's Encrypt</small>"]
            end

            subgraph frontend
                fe_app["Open-WebUI"]
                rag_web["Web Search Retrieval"]
                rag_doc["Document Store Retrieval"]
            end

            subgraph Ollama
                ollama["LLM API Endpoints"]
            end
        end
    end

    %% Flow
    user  -->|HTTPS| cf
    cf    -->|HTTPS over tunnel| firewall
    firewall -->|HTTPS| caddy
    caddy --> fe_app

    fe_app -.-> rag_web
    fe_app -.-> rag_doc
    fe_app -->|context + prompt| ollama

    fe_app -->|context + prompt| openai
```

## Background Problems and Technology Stack
1. Ollama is a framework for building and running LLM locally.
2. However running Ollama locally with a model on Docker Desktop is not practical.
3. Therefore, the solution is : `Bring-Your-Own-Model` with API key, such as OpenAI, OpenRouter, GitHub Models, etc.
4. Open-WebUI as a web-based LLM user interface. It has a built-in ChromaDB as default vector database.
5. Caddy as a reverse proxy. It is simple configuration but its default HTTPS makes backend services secure. 
6. CloudFlare Zero Trust allows public access to backend services over tunnel securely.
7. Tavily search for enabling Web Search to LLM
8. Docker Compose to wrap everything up as a portable YAML file
9. DevContainer lets us use a Docker container as a full-featured development environment.
10. GitHub Codespace as a container runtime for demo purpose

## Required Environment Variables for Cloudflare service
`TUNNEL_TOKEN` and `PUBLIC_HOSTNAME` must be set as Codespace secret variables or creating `.env` file in the root directory.