# Self-hosted Open-WebUI with Ollama on GitHub Codespace
Running Ollama with a model on Docker Desktop locally is a bad idea.

## Technology stack
1. Ollama is a framework for building and running LLM locally.
2. Open-WebUI as a web-based LLM interface
3. Caddy as a reverse proxy
4. CloudFlare Tunnels allows public access with HTTPS.
5. Bring-Your-Own-Model API key, such as OpenAI, OpenRouter, GitHub Models, etc.
6. Tavily search for enabling Web Search to LLM
7. Docker Compose to wrap everything a YAML file
8. DevContainer lets us use a Docker container as a full-featured development environment.
9. GitHub Codespace as a container runtime

## Conceptual Diagram
```mermaid
flowchart TD
    %% External actors
    subgraph Public
        user["fa:fa-user Prompt Engineer\n(web browser)"]
            subgraph OpenAI
                openai["LLM API Endpoints"]
            end
    end

    %% Cloudflare side
    subgraph CloudflareEdge
        cf["Cloudflare Zero Trust"]
    end

    %% Corporate network behind firewall
    subgraph Docker

        subgraph Bridge Network

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
                ollama["LLMs API Endpoints"]
            end
        end
    end

    %% Flow
    user  -->|HTTPS| cf
    cf    -->|HTTPS over tunnel| firewall
    firewall -->|HTTPS| caddy
    caddy --> fe_app

    fe_app --> rag_web
    fe_app --> rag_doc
    fe_app -.->|context + prompt| ollama

    fe_app -.->|context + prompt| openai
```
## Required Environment Variables for Cloudflare service
`TUNNEL_TOKEN` and `PUBLIC_HOSTNAME` must be set as Codespace secret variables or creating `.env` file in the root directory.