## 🧩 Part 11 Exercises (11.18 – 11.21)

### **11.18 Build success/failure notification**
I used the **discord-webhook-notify** GitHub Action to send messages to my Discord channel.  
It gives:  
- ✅ Success message when a new version is deployed  
- ❌ Error message when a build fails  

`.github/workflows/notify.yml`
```yaml
name: Notify on build result
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run build
        run: npm run build
      - name: Notify Discord success
        if: success()
        uses: Ilshidur/action-discord@v2
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }}
          message: "✅ New version deployed!"
      - name: Notify Discord failure
        if: failure()
        uses: Ilshidur/action-discord@v2
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }}
          message: "❌ Build failed on commit ${{ github.sha }}"

- name: Record start time
  run: echo "BUILD_START=$(date +%s)" >> $GITHUB_ENV

- name: Run build
  run: npm run build

- name: Show build time
  run: |
    END=$(date +%s)
    echo "Build took $((END - BUILD_START)) seconds"

- name: Report build metrics to Discord
  if: always()
  uses: Ilshidur/action-discord@v2
  with:
    webhook: ${{ secrets.DISCORD_WEBHOOK }}
    message: |
      📊 **Build Metrics**
      ⏱️ Duration: $((END - BUILD_START)) seconds
      🧱 Status: ${{ job.status }}

// buildHistory.js
import axios from "axios";

const GITHUB_API = "https://api.github.com/repos/Gladys334/Full-Stack-Open-Part-11/actions/runs";

async function getBuildHistory() {
  const response = await axios.get(GITHUB_API);
  const runs = response.data.workflow_runs.map(run => ({
    id: run.id,
    status: run.conclusion,
    created_at: run.created_at,
    updated_at: run.updated_at
  }));
  console.log("Recent Builds:", runs);
}

getBuildHistory();

