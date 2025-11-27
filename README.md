<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>TextShare — Social Text Platform</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
  <style>
    :root{
      --bg:#0f1724; /* deep slate */
      --card:#0b1220;
      --muted:#94a3b8;
      --accent:#7c3aed; /* purple */
      --glass: rgba(255,255,255,0.04);
      --radius:16px;
      --maxw:1100px;
      color-scheme: dark;
      font-family: 'Inter',system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#071024);color:#e6eef8}
    .wrap{max-width:var(--maxw);margin:40px auto;padding:28px}

    header{display:flex;align-items:center;justify-content:space-between;gap:16px}
    .brand{display:flex;gap:14px;align-items:center}
    .logo{width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;font-weight:800}
    nav{display:flex;gap:14px;align-items:center}
    nav a{color:var(--muted);text-decoration:none;padding:8px 12px;border-radius:10px}
    nav a.cta{background:linear-gradient(90deg,var(--accent),#06b6d4);color:white;font-weight:600}

    .hero{display:grid;grid-template-columns:1fr 420px;gap:28px;margin-top:32px;align-items:center}
    .hero-card{background:var(--card);padding:26px;border-radius:20px;box-shadow:0 6px 30px rgba(2,6,23,0.6)}
    h1{margin:0;font-size:clamp(26px,4vw,40px);line-height:1.02}
    p.lead{color:var(--muted);margin-top:10px}

    .buttons{display:flex;gap:12px;margin-top:18px}
    .btn{padding:10px 16px;border-radius:12px;background:var(--glass);border:1px solid rgba(255,255,255,0.03);cursor:pointer}
    .btn.primary{background:linear-gradient(90deg,var(--accent),#06b6d4);border:none;color:white;font-weight:700}

    .profile{display:flex;flex-direction:column;gap:14px;align-items:center;padding:20px;border-radius:16px;background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent)}
    .avatar{width:160px;height:160px;border-radius:20px;background:linear-gradient(135deg,#334155,#071024);display:flex;align-items:center;justify-content:center;font-size:40px;font-weight:700}
    .meta{display:flex;gap:12px}

    .grid{display:grid;grid-template-columns:repeat(3,1fr);gap:18px;margin-top:26px}
    .card{background:var(--card);padding:18px;border-radius:14px}

    footer{margin-top:36px;color:var(--muted);font-size:14px;text-align:center;padding:18px}

    /* Post styles */
    .post-form{display:grid;gap:12px;margin-bottom:24px}
    .post-input{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:14px;border-radius:10px;color:inherit;min-height:120px;resize:vertical;font-family:inherit}
    .post-input:focus{outline:none;border-color:var(--accent)}
    .post-actions{display:flex;justify-content:space-between;align-items:center}
    .char-count{color:var(--muted);font-size:14px}
    
    .posts-container{display:flex;flex-direction:column;gap:16px}
    .post{background:var(--card);padding:20px;border-radius:14px;border:1px solid rgba(255,255,255,0.04)}
    .post-header{display:flex;align-items:center;gap:10px;margin-bottom:12px}
    .post-avatar{width:36px;height:36px;border-radius:8px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;font-weight:700;font-size:14px}
    .post-author{font-weight:600}
    .post-time{color:var(--muted);font-size:13px}
    .post-content{line-height:1.5;margin-bottom:12px}
    .post-actions{display:flex;gap:16px}
    .post-action{display:flex;align-items:center;gap:6px;color:var(--muted);font-size:14px;cursor:pointer;transition:color 0.2s}
    .post-action:hover{color:#e6eef8}
    .post-action.liked{color:#f43f5e}
    
    /* responsive */
    @media (max-width:900px){
      .hero{grid-template-columns:1fr}
      .grid{grid-template-columns:repeat(1,1fr)}
      nav{display:none}
    }

    /* simple form styles */
    form{display:grid;gap:12px}
    input,textarea{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:10px;border-radius:10px;color:inherit}
    textarea{min-height:110px}
    .muted{color:var(--muted);font-size:14px}
    
    /* Modal styles */
    .modal{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.7);display:flex;align-items:center;justify-content:center;z-index:1000;padding:20px}
    .modal-content{background:var(--card);padding:24px;border-radius:16px;max-width:500px;width:100%;max-height:90vh;overflow-y:auto}
    .modal-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:16px}
    .modal-close{background:none;border:none;color:var(--muted);font-size:20px;cursor:pointer}
    .modal-close:hover{color:#e6eef8}
    
    /* Loading state */
    .loading{text-align:center;padding:20px;color:var(--muted)}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">
        <div class="logo">TS</div>
        <div>
          <div style="font-weight:700">TextShare</div>
          <div class="muted" style="font-size:13px">Share your thoughts — Read others'</div>
        </div>
      </div>

      <nav>
        <a href="#posts">Posts</a>
        <a href="#about">About</a>
        <a class="cta" href="#post" id="postBtn">New Post</a>
      </nav>
    </header>

    <main class="hero">
      <section class="hero-card">
        <h1>Share your thoughts with the world.</h1>
        <p class="lead">TextShare is a simple platform where you can post text and read what others have shared. No distractions, just words.</p>

        <div class="buttons">
          <button class="btn primary" id="startPostingBtn">Start Posting</button>
          <button class="btn" id="viewPostsBtn">View Posts</button>
        </div>

        <div class="grid" style="margin-top:22px">
          <div class="card">
            <strong>Real-time Updates</strong>
            <div class="muted">See posts from other users instantly.</div>
          </div>
          <div class="card">
            <strong>Anonymous</strong>
            <div class="muted">No sign-up required to read posts.</div>
          </div>
          <div class="card">
            <strong>Community</strong>
            <div class="muted">Share and discover thoughts from others.</div>
          </div>
        </div>
      </section>

      <aside class="profile">
        <div class="avatar">TS</div>
        <div style="text-align:center">
          <div style="font-weight:700">TextShare</div>
          <div class="muted">Powered by Firebase</div>
        </div>

        <div class="meta">
          <div class="muted">Posts: <span id="postCount">0</span></div>
          <div class="muted">Users: <span id="userCount">0</span></div>
        </div>
      </aside>
    </main>

    <section id="posts" style="margin-top:28px">
      <div class="hero-card">
        <h2>Recent Posts</h2>
        <div class="posts-container" id="postsContainer">
          <div class="loading">Loading posts...</div>
        </div>
      </div>
    </section>

    <section id="about" style="margin-top:18px">
      <div class="hero-card">
        <h2>About TextShare</h2>
        <p class="muted">TextShare is a minimalist platform for sharing text-based content. Post your thoughts, stories, ideas, or anything else you'd like to share with others. All posts are public and can be viewed by anyone visiting the site.</p>
        <p class="muted">Powered by Firebase for real-time updates across all users.</p>
      </div>
    </section>

    <footer>
      Built with ❤️ — Share your thoughts with the world.
    </footer>
  </div>

  <!-- Post Modal -->
  <div class="modal" id="postModal" style="display:none">
    <div class="modal-content">
      <div class="modal-header">
        <h3 style="margin:0">Create a New Post</h3>
        <button class="modal-close" id="closeModalBtn">&times;</button>
      </div>
      <form id="postForm">
        <input type="text" id="authorName" placeholder="Your name (optional)" maxlength="30" />
        <textarea id="postContent" class="post-input" placeholder="What's on your mind?" maxlength="500" required></textarea>
        <div class="post-actions">
          <div class="char-count"><span id="charCount">0</span>/500</div>
          <button class="btn primary" type="submit">Post</button>
        </div>
      </form>
    </div>
  </div>

  <script>
    // === REPLACE WITH YOUR FIREBASE CONFIG ===
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "123456789",
      appId: "YOUR_APP_ID"
    };

    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    // Generate a unique user ID
    function getUserId() {
      let userId = localStorage.getItem('textshare_user_id');
      if (!userId) {
        userId = 'user_' + Math.random().toString(36).substr(2, 9);
        localStorage.setItem('textshare_user_id', userId);
      }
      return userId;
    }

    // DOM elements
    const postsContainer = document.getElementById('postsContainer');
    const postModal = document.getElementById('postModal');
    const postForm = document.getElementById('postForm');
    const postContent = document.getElementById('postContent');
    const authorName = document.getElementById('authorName');
    const charCount = document.getElementById('charCount');
    const closeModalBtn = document.getElementById('closeModalBtn');
    const postBtn = document.getElementById('postBtn');
    const startPostingBtn = document.getElementById('startPostingBtn');
    const viewPostsBtn = document.getElementById('viewPostsBtn');
    const postCount = document.getElementById('postCount');
    const userCount = document.getElementById('userCount');

    let posts = [];
    let likes = {};

    // Initialize the app
    function init() {
      const userId = getUserId();
      
      // Set up real-time listener for posts
      db.collection('posts')
        .orderBy('timestamp', 'desc')
        .onSnapshot((snapshot) => {
          posts = [];
          snapshot.forEach(doc => {
            posts.push({ id: doc.id, ...doc.data() });
          });
          renderPosts();
          updateStats();
        });

      // Set up real-time listener for likes
      db.collection('likes').doc('global')
        .onSnapshot((doc) => {
          likes = doc.exists ? doc.data() : {};
        });

      // Event listeners
      postBtn.addEventListener('click', openPostModal);
      startPostingBtn.addEventListener('click', openPostModal);
      viewPostsBtn.addEventListener('click', () => {
        document.getElementById('posts').scrollIntoView({behavior: 'smooth'});
      });
      closeModalBtn.addEventListener('click', closePostModal);
      postForm.addEventListener('submit', handlePostSubmit);
      postContent.addEventListener('input', updateCharCount);
      
      // Close modal when clicking outside
      window.addEventListener('click', (e) => {
        if (e.target === postModal) {
          closePostModal();
        }
      });
    }

    // Render all posts
    function renderPosts() {
      const userId = getUserId();
      
      postsContainer.innerHTML = '';
      
      if (posts.length === 0) {
        postsContainer.innerHTML = '<p class="muted" style="text-align:center;padding:20px">No posts yet. Be the first to share!</p>';
        return;
      }
      
      posts.forEach(post => {
        // Check if current user has liked this post
        const userLiked = likes[post.id] && likes[post.id].includes(userId);
        const postElement = createPostElement(post, userLiked);
        postsContainer.appendChild(postElement);
      });
    }

    // Create a post element
    function createPostElement(post, userLiked) {
      const postDiv = document.createElement('div');
      postDiv.className = 'post';
      postDiv.dataset.id = post.id;
      
      const timeAgo = getTimeAgo(post.timestamp);
      const authorInitial = post.author ? post.author.charAt(0).toUpperCase() : 'A';
      
      postDiv.innerHTML = `
        <div class="post-header">
          <div class="post-avatar">${authorInitial}</div>
          <div>
            <div class="post-author">${post.author || 'Anonymous'}</div>
            <div class="post-time">${timeAgo}</div>
          </div>
        </div>
        <div class="post-content">${post.content}</div>
        <div class="post-actions">
          <div class="post-action ${userLiked ? 'liked' : ''}" onclick="likePost('${post.id}')">
            <span>❤️</span>
            <span>${post.likes || 0}</span>
          </div>
        </div>
      `;
      
      return postDiv;
    }

    // Handle post submission
    async function handlePostSubmit(e) {
      e.preventDefault();
      
      const content = postContent.value.trim();
      const author = authorName.value.trim();
      
      if (!content) return;
      
      const newPost = {
        author: author || null,
        content: content,
        timestamp: firebase.firestore.FieldValue.serverTimestamp(),
        likes: 0
      };
      
      try {
        await db.collection('posts').add(newPost);
        closePostModal();
        postForm.reset();
        updateCharCount();
      } catch (error) {
        console.error('Error adding post:', error);
        alert('Error posting. Please try again.');
      }
    }

    // Like a post
    async function likePost(postId) {
      const userId = getUserId();
      
      // Initialize likes array for this post if it doesn't exist
      if (!likes[postId]) {
        likes[postId] = [];
      }
      
      // Check if user already liked this post
      const userLiked = likes[postId].includes(userId);
      
      try {
        if (userLiked) {
          // Unlike the post
          await db.collection('posts').doc(postId).update({
            likes: firebase.firestore.FieldValue.increment(-1)
          });
          likes[postId] = likes[postId].filter(id => id !== userId);
        } else {
          // Like the post
          await db.collection('posts').doc(postId).update({
            likes: firebase.firestore.FieldValue.increment(1)
          });
          likes[postId].push(userId);
        }
        
        // Update likes in Firestore
        await db.collection('likes').doc('global').set(likes);
        
      } catch (error) {
        console.error('Error updating likes:', error);
      }
    }

    // Open post modal
    function openPostModal() {
      postModal.style.display = 'flex';
      postContent.focus();
    }

    // Close post modal
    function closePostModal() {
      postModal.style.display = 'none';
    }

    // Update character count
    function updateCharCount() {
      charCount.textContent = postContent.value.length;
    }

    // Calculate time ago
    function getTimeAgo(timestamp) {
      if (!timestamp) return 'Recently';
      
      const now = new Date();
      const postTime = timestamp.toDate();
      const diffMs = now - postTime;
      const diffMins = Math.floor(diffMs / 60000);
      const diffHours = Math.floor(diffMs / 3600000);
      const diffDays = Math.floor(diffMs / 86400000);
      
      if (diffMins < 1) return 'Just now';
      if (diffMins < 60) return `${diffMins} min ago`;
      if (diffHours < 24) return `${diffHours} hour${diffHours > 1 ? 's' : ''} ago`;
      return `${diffDays} day${diffDays > 1 ? 's' : ''} ago`;
    }

    // Update statistics
    function updateStats() {
      postCount.textContent = posts.length;
      
      // Count unique authors
      const authors = new Set(posts.map(post => post.author).filter(author => author));
      userCount.textContent = authors.size;
    }

    // Initialize the app when DOM is loaded
    document.addEventListener('DOMContentLoaded', init);
  </script>
</body>
</html>
