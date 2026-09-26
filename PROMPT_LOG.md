# Prompt Log

## AI tools used
- **ChatGPT / Codex:** built the original Miao app for HW3: the FastAPI backend, the OpenAI food detection, meal history and PDF reports.
- **Claude:** planned the HW4 changes: deploying on Render, separating the backend into its own repo, restricting CORS, securing the API key, and connecting the GitHub Pages frontend.

## Key prompts that shaped the implementation

### Deploying for HW4 (Claude)
- Ok, break down the steps for building a complete backend. Here are the files for what my current health tracker looks like. I am working in Visual Studio code.
- One problem you have to fix first. The last line of main.py is app.mount("/", StaticFiles(directory=ROOT, html=True)). Once deployed, that serves every file in the backend folder to anyone, including main.py and the saved-meals database. Step 1 removes it.
- Phase 1: Create the backend repo (in VS Code)
Make a new folder, e.g. miao-backend, and open it in VS Code (File → Open Folder).
Copy in only main.py and requirements.txt. The HTML, CSS and JS files stay in your portfolio repo.
Edit main.py:
Restrict CORS (lines 33–38) so only your sites can call the backend:
python
     app.add_middleware(
       CORSMiddleware,
       allow_origins=["https://sharon5u.github.io", "http://127.0.0.1:5500"],
       allow_methods=["*"],
       allow_headers=["*"],
     )
 The second address is for testing locally with VS Code's Live Server extension.
Replace the bottom of the file (everything from @app.get("/") on line 846 to the end) with a simple status check:
python
     @app.get("/")
     async def health():
       return {"status": "ok", "service": "miao backend"}
 This removes the file-serving line. It also gives you a URL that shows the backend is running, which is handy in the video.

4. Create a .gitignore file:

   .env
   .venv/
   __pycache__/
   *.sqlite3
Test it locally in VS Code's terminal (Terminal → New Terminal):
bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   export OPENAI_API_KEY="your_key_here"
   uvicorn main:app --reload

Open http://127.0.0.1:8000/docs. FastAPI generates a test page for every endpoint there. Try /api/detect-food with a food photo and check that it returns JSON.
6. Publish it: open Source Control (the branch icon in the left bar), commit, then click Publish Branch and choose public. Before you publish, confirm that .env and .venv are not in the list of files being committed.

Phase 2: Deploy on Render
Go to render.com and click New → Web Service, then connect the miao-backend repo. Use these settings:
Runtime: Python 3
Build Command: pip install -r requirements.txt
Start Command: uvicorn main:app --host 0.0.0.0 --port $PORT
Instance type: Free
Environment variable: OPENAI_API_KEY = your key
Once the deploy finishes, open your Render URL. You should see {"status":"ok",...}. If food detection later fails with a model error, add an OPENAI_MODEL environment variable in Render with a model your key can use.
Phase 3: Point the frontend at Render (portfolio repo)
Open your sharon5u.github.io folder in VS Code and edit Health tracker/app.js:
Line 52: const API_BASE = "https://YOUR-APP.onrender.com";
Line 1100: change the "Could not reach the local app server…" message to something like "Could not reach the server. It may be waking up, so wait 30 seconds and try again."
In Health tracker/service-worker.js, change miao-v33 to miao-v34. This makes browsers download the new app.js instead of an old saved copy.
Delete main.py and requirements.txt from the Health tracker/ folder, since they now live in the backend repo.
In the portfolio's main index.html, change the Miao card's "View Project" link to https://sharon5u.github.io/Health%20tracker/.
Commit and push, then open the live page and test: upload a photo, scan it, save the meal, and check that the meal history loads.

- when you say edit main.py in step 3, do you mean editing the original or the one in the backend
- Edit the copy in Miao-backend. Leave the original exactly as it is.

- moving onto phase 2, Do I need to upload my miao-backend repo to github to do step 7?
- Yes. Render deploys straight from a GitHub repo, so publish miao-backend first. That's step 6, the last step of Phase 1.

To publish from VS Code:

Stop the local server if it's still running (press Ctrl + C in the terminal).
Click the Source Control icon in the left bar (the branch-shaped one).
Click Initialize Repository.
Before committing, check the file list. You should see only .gitignore, main.py and requirements.txt. If .venv or any .env file appears, stop: the .gitignore isn't working, so tell me before going further.
Type a message like Initial backend in the box, then click Commit. If it asks whether to stage all changes, click Yes.
Click Publish Branch, sign in to GitHub if it asks, and choose Publish to GitHub public repository. Keep the name miao-backend, or pick your own.
Open github.com/sharon5u/miao-backend in your browser and confirm the three files are there, with no .venv folder.

Then do step 7. On Render, click New → Web Service, and miao-backend will appear in your repo list. If it doesn't, click Configure account and give Render access to that repo.

The README and prompt log (Phase 4) aren't needed yet. You can add them to the repo later, and Render will redeploy automatically each time you push.

That copy is the one that goes to GitHub and Render, so it needs both edits (restricting CORS, and replacing the bottom of the file). The original stays as your backup until you delete it in step 11.

To be sure you're editing the right file in VS Code, open the Miao-backend folder itself (File → Open Folder → Miao-backend). Then the only main.py in the file list is the copy.

- Ok, now the backend and front end are on different repositories, is that right? My previous health tracker in inside my github.io repo and under a folder called health tracker
- Yes, that's the right setup. miao-backend is published as a public repo, which is what the assignment asks for. The frontend stays where it already is, in the Health tracker folder of your sharon5u.github.io repo. That's also what makes it show up on GitHub Pages, at https://sharon5u.github.io/Health%20tracker/.

	Repo	Where it runs
Backend	miao-backend	Render
Frontend	sharon5u.github.io → Health tracker/	GitHub Pages

In Phase 3, edit the frontend inside the sharon5u.github.io repo, not in miao-github-upload-newestAAA. That other folder is just a local copy and isn't connected to your website. If it has a newer version of the frontend files than the repo does, copy just the HTML, JS, CSS, manifest and PNG files into Health tracker/ at that point.

Also in Phase 3, delete main.py, requirements.txt and the stray nono file from the Health tracker folder in the sharon5u.github.io repo, since the backend now has its own repo.

For now, go ahead with step 7 on Render and connect the miao-backend repo.
