9JATUNES AUTO-STORIES: SETUP IN PLAIN WORDS
===========================================

WHAT THIS DOES
Every 3 hours, GitHub quietly visits the public news feeds of Nigerian
entertainment websites, picks out the newest headlines, and saves them in a file
called stories.json. Your website reads that file and shows the stories by itself,
sorted into Music, Celebrities, Gist, Movies and Sport. Stories that mention Asake,
Burna Boy, Seyi Vibez, Tiwa Savage and the other stars also appear on their profiles.

Each automatic story shows only the headline, a short summary, a small picture and a
"Read the full story" link to the original website.

HOW TO SET IT UP (one time, about 5 minutes)
1. Open your 9jatunes repository on GitHub.
2. Upload everything in this folder into the main (top) level of the repository.
   - This replaces your old index.html with the new one.
   - Make sure the hidden folder named ".github" goes up too. If your computer or
     browser hides it, open "Add file > Create new file" on GitHub, type
     .github/workflows/update-stories.yml as the name, and paste in the text from
     that file.
3. Go to Settings > Actions > General. Scroll to "Workflow permissions", choose
   "Read and write permissions", then press Save.
4. Go to the Actions tab, click "Update stories" on the left, then press
   "Run workflow". Wait about a minute.
5. Click the finished run to see its log. Each news site shows OK or SKIP:
   - OK means its stories were collected.
   - SKIP means that site did not answer. You can ignore it or remove it
     (see "Changing the news sites" below).
6. Refresh your website. A "Live feed" label appears above the stories, and the
   new stories show up under the "Live feed" button. After this it updates itself.

CHANGING THE NEWS SITES
Open sources.json on GitHub, click the pencil icon, and add, remove or edit a site
(each site has a name and its feed link). Press "Commit changes".

TURNING OFF THE SMALL PICTURES
In sources.json, change   "useSourceImages": true   to   false.
Stories then show a celebrity photo from Wikipedia, or a plain colour tile.

GOOD TO KNOW
- Headlines, summaries and pictures belong to the original websites. That is why
  each story links back to its source. Please confirm this approach with the client.
- The stories written by hand stay exactly as they are. Automatic ones are added
  beside them, and anything older than 3 weeks drops off by itself.
- If the repository has no activity for a long time, GitHub may pause the timer.
  Just open the Actions tab and switch it back on.
- This setup assumes your main branch is called "main".
