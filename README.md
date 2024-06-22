# Push-to-Deploy for Hosting.de

This GitHub Action brings an entire push-to-deploy workflow to your project.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    concurrency: deploy-${{ matrix.app }}-${{ github.ref_name }}
    strategy:
      fail-fast: false
      matrix:
        # All keys under `applications` in the manifest
        app:
          - app1
          - app2
    environment:
      name: "${{ github.ref_name }}-${{ matrix.app }}"
      url: ${{ steps.deploy.outputs.public-url }}
    steps:
      - name: Deploy
        id: deploy
        uses: richardhj/deploy-hostingde@main
        with:
          auth-token: ${{ secrets.HOSTINGDE_AUTH_TOKEN }}
          app: ${{ matrix.app }}
          project-prefix: 'project1'
          ssh-public-key: ${{ secrets.SSH_PUBLIC_KEY }}
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          dotenv: |
            MAILER_DSN=${{ secrets.MAILER_DSN }}
            SOME_API_KEY=${{ secrets.SOME_API_KEY }}
          php-version: 8.3
          # Example for a Symfony project
          build-hook: |
              set -x -e
              composer install --prefer-dist --no-scripts --no-progress --no-ansi --no-interaction
              curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/download/v3.4.3/tailwindcss-linux-x64
              mv tailwindcss-linux-x64 bin/tailwindcss
              chmod +x bin/tailwindcss
              ./bin/tailwindcss -i www.css -o www.tailwind.css -c tailwind-www.config.js --minify
              php bin/console importmap:install
              php bin/console asset-map:compile
```
