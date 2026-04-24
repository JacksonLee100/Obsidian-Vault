
uv run uvicorn server:app --host 0.0.0.0 --port 8082

$env:ANTHROPIC_BASE_URL="http://localhost:1234"
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
$env:ANTHROPIC_AUTH_TOKEN="freecc"; $env:ANTHROPIC_BASE_URL="http://localhost:8082"; claude