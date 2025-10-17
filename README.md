# AI Digital Twin

A full-stack AI chatbot application powered by AWS Bedrock, built with FastAPI backend and Next.js frontend, deployed on AWS infrastructure using Terraform.

## Overview

This project creates a conversational AI "digital twin" that maintains conversation history and can be customized with different personalities or contexts. The application leverages AWS Bedrock's Nova models for natural language understanding and generation.

## Architecture

- **Frontend**: Next.js (React) with TypeScript, deployed as static site on S3 + CloudFront
- **Backend**: FastAPI (Python) with AWS Lambda + API Gateway
- **AI**: AWS Bedrock (Amazon Nova models)
- **Storage**: S3 for conversation memory
- **Infrastructure**: Terraform for IaC

## Features

- Real-time chat interface with AI assistant
- Persistent conversation memory (session-based)
- Configurable AI personality through system prompts
- Local and cloud storage options for conversations
- Serverless architecture for cost efficiency
- CORS-enabled API for secure cross-origin requests

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI configured with credentials
- Terraform >= 1.0
- Node.js >= 18.x
- Python >= 3.9
- Access to AWS Bedrock Nova models (enable in your AWS region)

## Project Structure

```
.
├── backend/              # FastAPI application
│   ├── server.py        # Main API server
│   ├── lambda_handler.py # AWS Lambda entry point
│   ├── context.py       # AI system prompt configuration
│   └── resources.py     # Additional backend resources
├── frontend/            # Next.js application
│   ├── app/            # Next.js app directory
│   ├── components/     # React components
│   └── public/         # Static assets
├── terraform/           # Infrastructure as Code
│   ├── main.tf         # Main infrastructure
│   ├── backend-setup.tf # Terraform state backend (run once)
│   ├── variables.tf    # Variable definitions
│   └── outputs.tf      # Output values
├── scripts/             # Deployment scripts
│   ├── deploy.sh       # Deploy infrastructure
│   └── destroy.sh      # Tear down infrastructure
└── memory/             # Local conversation storage (development)
```

## Setup Instructions

### 1. Clone and Configure

```bash
git clone <repository-url>
cd twin

# Copy environment template
cp .env.example .env

# Edit .env with your AWS account details
nano .env
```

### 2. Configure Environment Variables

Edit [.env](.env) and set:

```bash
AWS_ACCOUNT_ID=your_12_digit_account_id
DEFAULT_AWS_REGION=us-east-1  # or your preferred region
PROJECT_NAME=twin
```

### 3. Set Up Terraform Backend (First Time Only)

```bash
cd terraform

# Initialize and create state backend
terraform init
terraform apply -target=aws_s3_bucket.terraform_state -target=aws_dynamodb_table.terraform_locks

# After successful creation, you can optionally remove backend-setup.tf
# Then uncomment the backend configuration in versions.tf
```

### 4. Deploy Infrastructure

```bash
# From project root
./scripts/deploy.sh

# This will:
# - Build the frontend
# - Package the backend
# - Deploy via Terraform
# - Output the application URLs
```

### 5. Access Your Application

After deployment completes, Terraform will output:

```
cloudfront_url = "https://xxxxx.cloudfront.net"
api_gateway_url = "https://xxxxx.execute-api.region.amazonaws.com"
```

Visit the CloudFront URL to use your AI Digital Twin!

## Local Development

### Backend Development

```bash
cd backend

# Install dependencies
pip install fastapi uvicorn python-dotenv boto3

# Run locally (uses local file storage for memory)
python server.py

# API available at http://localhost:8000
# API docs at http://localhost:8000/docs
```

### Frontend Development

```bash
cd frontend

# Install dependencies
npm install

# Run development server
npm run dev

# Frontend available at http://localhost:3000
```

## Configuration

### AI Model Selection

Edit [backend/server.py](backend/server.py#L41) to choose your Bedrock model:

```python
# Available models:
# - amazon.nova-micro-v1:0  (fastest, cheapest)
# - amazon.nova-lite-v1:0   (balanced - default)
# - amazon.nova-pro-v1:0    (most capable, higher cost)
BEDROCK_MODEL_ID = "amazon.nova-lite-v1:0"
```

Note: Some regions may require a prefix (us. or eu.) before the model ID.

### Customize AI Personality

Edit [backend/context.py](backend/context.py) to modify the system prompt and change how your digital twin behaves.

### Storage Configuration

The backend supports two storage modes:

- **Local**: Files stored in [memory/](memory/) directory (development)
- **S3**: Conversations stored in S3 bucket (production)

Set via environment variable `USE_S3=true` or `USE_S3=false`

## API Endpoints

- `GET /` - API information and status
- `GET /health` - Health check endpoint
- `POST /chat` - Send message and get AI response
- `GET /conversation/{session_id}` - Retrieve conversation history

## Cleanup

To destroy all AWS resources:

```bash
./scripts/destroy.sh
```

Warning: This will delete all deployed resources including conversation data stored in S3.

## Cost Considerations

- **AWS Bedrock**: Pay per token (input/output)
- **Lambda**: Generous free tier, then pay per request
- **S3**: Very low cost for storage
- **CloudFront**: Free tier includes 1TB/month for 12 months
- **API Gateway**: Free tier includes 1M requests/month for 12 months

Estimated cost for low-moderate usage: $5-20/month

## Troubleshooting

### Bedrock Access Denied

Enable model access in AWS Console:
1. Go to AWS Bedrock console
2. Navigate to "Model access"
3. Request access to Amazon Nova models

### CORS Issues

Update `CORS_ORIGINS` in [.env](.env) to include your frontend URL.

### Lambda Package Size

If lambda deployment fails due to package size, ensure you're only packaging necessary dependencies in [backend/lambda-package/](backend/lambda-package/).

## Security Notes

- API Gateway endpoints are public by default
- Consider adding authentication for production use
- Never commit [.env](.env) file (already in [.gitignore](.gitignore))
- S3 buckets are configured with encryption and public access blocks

## License

MIT

## Contributing

Contributions welcome! Please open an issue or submit a pull request.
