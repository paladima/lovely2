<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>LovelyApp</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    body {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
      background-attachment: fixed !important;
      margin: 0 !important;
      min-height: 100vh !important;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    }
    .hidden { display: none !important; }
    .navbar {
      position: fixed; top: 0; left: 0; right: 0; background: rgba(255,255,255,0.98);
      backdrop-filter: blur(15px); padding: 15px 20px; box-shadow: 0 2px 20px rgba(0,0,0,0.1);
      z-index: 1000; display: flex; justify-content: space-between; align-items: center;
      border-bottom: 1px solid #f0f0f0;
    }
    .navbar-left { display: flex; align-items: center; gap: 20px; }
    .logo { display: flex; align-items: center; gap: 8px; font-size: 24px; font-weight: bold; color: #e91e63; }
    .logo-icon {
      width: 32px; height: 32px; background: linear-gradient(45deg, #e91e63, #ff6b9d);
      border-radius: 50%; display: flex; align-items: center; justify-content: center;
      color: white; font-size: 18px;
    }
    .auth-buttons { display: flex; gap: 10px; }
    .auth-btn {
      padding: 8px 15px; border: none; border-radius: 20px; cursor: pointer;
      font-size: 14px; font-weight: bold; transition: all 0.3s ease;
    }
    .login-btn { background: linear-gradient(to right, #5ecbff, #4db8e8); color: white; }
    .logout-btn { background: linear-gradient(to right, #ff9ecf, #ff7ab8); color: white; }
    .main-container { margin-top: 80px; padding: 20px; max-width: 1400px; margin: 80px auto 0 auto; }
    .dating-page, .edit-profile-page, .view-profile-page { display: flex; gap: 30px; }
    .sidebar-profile {
      flex: 0 0 280px; background: rgba(255,255,255,0.95); backdrop-filter: blur(15px);
      border-radius: 20px; padding: 30px 20px; box-shadow: 0 8px 32px rgba(0,0,0,0.1);
      height: fit-content; position: sticky; top: 100px;
    }
    .sidebar-avatar-container { position: relative; width: 120px; height: 120px; margin: 0 auto 20px; }
    .sidebar-avatar {
        width: 100%; height: 100%; border-radius: 50%;
        display: flex; align-items: center; justify-content: center; font-size: 48px; color: white;
        border: 4px solid rgba(255,255,255,0.8); background: linear-gradient(135deg, #ff9ecf, #ffc3e8);
        background-size: cover; background-position: center; background-repeat: no-repeat;
    }
    .upload-avatar-overlay {
        position: absolute; bottom: 0; left: 0; width: 100%; background: rgba(0,0,0,0.4);
        color: white; text-align: center; padding: 5px 0; font-size: 12px;
        opacity: 0; transition: opacity 0.3s; cursor: pointer; border-bottom-left-radius: 60px; border-bottom-right-radius: 60px;
    }
    .sidebar-avatar-container:hover .upload-avatar-overlay { opacity: 1; }
    .sidebar-name { text-align: center; font-size: 24px; font-weight: bold; color: #333; margin-bottom: 5px; }
    .sidebar-location { text-align: center; color: #666; font-size: 14px; margin-bottom: 20px; }
    .sidebar-actions { display: flex; flex-direction: column; gap: 10px; }
    .sidebar-btn { padding: 12px 15px; border: none; border-radius: 12px; font-weight: 600; cursor: pointer; font-size: 14px; }
    .sidebar-btn.primary { background: linear-gradient(to right, #ff9ecf, #ff7ab8); color: white; }
    .sidebar-btn.secondary { background: rgba(233, 30, 99, 0.1); color: #e91e63; }
    .main-content {
      flex: 1; background: rgba(255,255,255,0.95); backdrop-filter: blur(15px);
      border-radius: 20px; padding: 30px; box-shadow: 0 8px 32px rgba(0,0,0,0.1);
    }
    .page-title { font-size: 32px; font-weight: bold; margin-bottom: 30px; }
    .users-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 25px; }
    .user-card { background: white; border-radius: 20px; overflow: hidden; box-shadow: 0 8px 32px rgba(0,0,0,0.1); }
    .user-photo { width: 100%; height: 280px; display: flex; align-items: center; justify-content: center; font-size: 48px; color: white; font-weight: bold; background-size: cover; background-position: center; cursor: pointer; }
    .user-info { padding: 20px; text-align: center; }
    .user-name { font-size: 18px; font-weight: bold; color: #333; }
    .user-location { font-size: 13px; color: #666; margin-top: 5px; }
    .user-actions { display: flex; justify-content: center; margin-top: 15px; }
    .action-btn-circle { width: 50px; height: 50px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 20px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
    .btn-message { background: #e0f7fa; color: #00bcd4; }
    .form-box {
        background: rgba(255,255,255,0.95); backdrop-filter: blur(15px); padding: 30px;
        border-radius:20px; width:100%; max-width: 400px; margin: 20px auto;
        box-shadow: 0 8px 32px rgba(0,0,0,0.2); box-sizing: border-box;
    }
    .form-box h2 { text-align:center; margin-bottom:25px; color:#333; }
    .form-box .form-group { margin-bottom: 15px; }
    .form-box label { font-size: 14px; color: #666; margin-bottom: 8px; display: block; }
    .form-box input, .form-box select, .form-box textarea { width:100%; padding:12px; border:none; border-radius:8px; background:#f5f5f5; color:#333; font-size:14px; box-sizing:border-box; font-family: inherit; }
    .form-box .form-toggle {text-align: center; margin-top: 20px; font-size: 14px;}
    .form-box .toggle-link { color: #e91e63; cursor: pointer; font-weight: 600; }
    .btn-submit { width: 100%; background: linear-gradient(to right, #5ecbff, #4db8e8); color: #fff; padding: 14px 10px; border:none; border-radius:10px; font-weight:bold; font-size:16px; cursor:pointer; margin-top: 10px; }
    .modal {
        position: fixed; z-index: 2000; left: 0; top: 0;
        width: 100%; height: 100%; background-color: rgba(0, 0, 0, 0.5);
        backdrop-filter: blur(5px);
        align-items: center; justify-content: center;
        display: none; overflow-y: auto; padding: 20px;
    }
    .modal.is-open { display: flex; }
    .modal-content {
        background: white; padding: 30px; border-radius: 20px;
        width: 100%; max-width: 500px;
        box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        position: relative; margin: auto;
    }
    .modal-close-btn {
        position: absolute; top: 15px; right: 15px;
        width: 30px; height: 30px; border-radius: 50%;
        border: none; background: #f0f0f0; color: #888;
        cursor: pointer; font-size: 16px; line-height: 30px;
        text-align: center; z-index: 10;
    }
    .photo-gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(120px, 1fr)); gap: 10px; margin-top: 15px; padding-top: 15px; border-top: 1px solid #f0f0f0; }
    .gallery-item { position: relative; cursor: pointer;}
    .gallery-img { width: 100%; height: 120px; object-fit: cover; border-radius: 8px; }
    .delete-photo-btn { position: absolute; top: -5px; right: -5px; width: 22px; height: 22px; border-radius: 50%; border: 1px solid white; background: rgba(0,0,0,0.6); color: white; cursor: pointer; font-size: 12px; line-height: 20px; text-align: center; }
    .main-photo-check { position: absolute; bottom: 5px; left: 5px; width: 22px; height: 22px; background: rgba(255,255,255,0.8); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: #28a745; font-weight: bold; font-size: 16px; }
    .add-photo-card { border: 2px dashed #ccc; border-radius: 12px; display: flex; align-items: center; justify-content: center; height: 120px; cursor: pointer; color: #ccc; font-size: 48px; flex-direction: column; font-weight: normal; }
    .add-photo-card span { font-size: 14px; margin-top: 5px; }
    .profile-card { background: rgba(255,255,255,0.95); backdrop-filter: blur(15px); border-radius: 20px; padding: 30px; box-shadow: 0 8px 32px rgba(0,0,0,0.1); }
    .profile-section { margin-top: 30px; }
    .profile-section-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #f0f0f0; padding-bottom: 10px; margin-bottom: 15px; }
    .profile-section h3 { font-size: 18px; margin: 0; }
    .info-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
    .back-to-nearby { margin-bottom: 20px; cursor: pointer; color: #e91e63; font-weight: bold; }
    .view-profile-page { display: block; }
    .profile-header { text-align: center; margin-bottom: 20px; }
    .profile-main-photo { width: 150px; height: 150px; border-radius: 50%; object-fit: cover; background-color: #f0f0f0; margin: 0 auto 20px; border: 4px solid white; box-shadow: 0 4px 15px rgba(0,0,0,0.1); display: flex; align-items: center; justify-content: center; font-size: 72px; color: white; background-size: cover; background-position: center;}
    .profile-name { font-size: 28px; font-weight: bold; }
    .profile-location { font-size: 16px; color: #666; }
    .profile-actions { display: flex; justify-content: center; gap: 15px; margin-top: 20px; }
    .profile-action-btn { padding: 12px 25px; border-radius: 25px; border: none; font-weight: bold; cursor: pointer; text-align: center; }
    .profile-action-btn.chat { background-color: #00bcd4; color: white; }
    .profile-action-btn.edit { background-color: #6c757d; color: white; }
    .info-item { background: #f7f7f7; padding: 10px; border-radius: 8px; }
    .info-label { font-size: 12px; color: #888; display: block; margin-bottom: 4px; }
    .info-value { font-size: 14px; font-weight: 600; }
    .interests-container { display: flex; flex-wrap: wrap; gap: 10px; }
    .interest-tag { background: #e0e0e0; padding: 5px 12px; border-radius: 15px; font-size: 14px; }
    .profile-gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 10px; }
    .profile-gallery-img { width: 100%; height: 150px; object-fit: cover; border-radius: 12px; }
    .chat-modal-content {
      background-color: #fce4ec; 
      width: 400px; max-width: 90%; height: 80vh; max-height: 700px;
      border-radius: 20px;
      box-shadow: 0 10px 40px rgba(0,0,0,0.2); display: flex;
      flex-direction: column; overflow: hidden;
      margin: auto;
    }
    .chat-header { padding: 15px 20px; background: rgba(255, 255, 255, 0.7); display: flex; align-items: center; gap: 15px; border-bottom: 1px solid #f8bbd0; }
    .chat-avatar { width: 40px; height: 40px; border-radius: 50%; background: linear-gradient(135deg, #ff9ecf, #ffc3e8); color: white; font-size: 20px; font-weight: bold; display: flex; align-items: center; justify-content: center; background-size: cover; background-position: center;}
    .chat-user-info .name { font-weight: bold; font-size: 16px; color: #333; }
    .chat-close-btn { margin-left: auto; width: 30px; height: 30px; border-radius: 50%; border: none; background: #fce4ec; color: #ad1457; cursor: pointer; font-size: 16px; }
    .chat-messages-area { flex-grow: 1; padding: 20px; overflow-y: auto; display: flex; flex-direction: column; gap: 12px; }
    .message-bubble { max-width: 75%; padding: 10px 15px; border-radius: 20px; font-size: 15px; word-wrap: break-word; }
    .message-bubble.sent { background-color: #ff80ab; color: white; border-bottom-right-radius: 5px; align-self: flex-end; }
    .message-bubble.received { background-color: #ffffff; color: #333; border-bottom-left-radius: 5px; align-self: flex-start; }
    .message-attachment { max-width: 100%; border-radius: 15px; margin-top: 5px; }
    .message-bubble.has-attachment { padding: 5px; }
    .chat-form { display: flex; padding: 15px; background: rgba(255, 255, 255, 0.7); border-top: 1px solid #f8bbd0; gap: 10px; }
    #chatMessageInput { flex-grow: 1; border: none; background: #fff; border-radius: 20px; padding: 0 15px; font-size: 15px; }
    #chatMessageInput:focus { outline: none; }
    .chat-form button { width: 45px; height: 45px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 20px; }
    #sendMessageBtn { background: #ff80ab; color: white; }
    #uploadBtn { background: #f0f0f0; color: #555; }
  </style>
</head>
<body>

<div class="navbar">
    <div class="navbar-left">
        <div class="logo"><div class="logo-icon">♥</div><span>LovelyApp</span></div>
        <div class="navbar-nav hidden" id="mainNav"><div class="nav-item active">Nearby</div></div>
    </div>
    <div class="navbar-right">
        <div class="auth-buttons">
            <button class="auth-btn login-btn" id="loginNavBtn">Login</button>
            <button class="auth-btn logout-btn hidden" id="logoutBtn">Logout</button>
        </div>
    </div>
</div>

<div class="main-container">
    <div id="registerSection" class="form-box hidden">
        <h2>Ready to Feel the Spark?</h2>
        <form id="registerForm">
            <div class="form-group"><input type="text" id="registerName" placeholder="Your Name" required></div>
            <div class="form-group"><input type="email" id="registerEmail" placeholder="E-mail" required autocomplete="email"></div>
            <div class="form-group"><input type="password" id="registerPassword" placeholder="Password" required minlength="6" autocomplete="new-password"></div>
            <div class="form-group"><input type="number" id="registerAge" placeholder="Age" required min="18"></div>
            <div class="form-group"><input type="text" id="registerCountry" placeholder="Country" required></div>
            <div class="form-group">
                <label for="registerGender">Gender</label>
                <select id="registerGender" required><option value="Man">Man</option><option value="Woman">Woman</option></select>
            </div>
            <button type="submit" class="btn-submit">Register</button>
            <div class="form-toggle"><span>Already have an account? <a href="#" id="showLoginFormLink">Sign in</a></span></div>
        </form>
    </div>
    <div id="loginSection" class="form-box hidden">
        <h2>Welcome Back!</h2>
        <form id="loginForm">
            <div class="form-group"><input type="email" id="loginEmail" placeholder="E-mail" required autocomplete="email"></div>
            <div class="form-group"><input type="password" id="loginPassword" placeholder="Password" required autocomplete="current-password"></div>
            <button type="submit" class="btn-submit">Login</button>
            <div class="form-toggle"><span>New here? <a href="#" id="showRegisterFormLink">Create account</a></span></div>
        </form>
    </div>

    <div id="profileSection" class="hidden">
      <div class="dating-page">
          <div class="sidebar-profile" id="mainSidebar"></div>
          <div class="main-content">
              <h1 class="page-title">Nearby</h1>
              <div class="users-grid" id="usersGrid"></div>
          </div>
      </div>
    </div>
    
    <div id="viewProfilePageSection" class="hidden">
        <div class="back-to-nearby" id="viewProfileBackBtn">← Back to Nearby</div>
        <div class="profile-card" style="max-width: 900px; margin: 0 auto;">
            <div class="profile-header">
                <div id="profileMainPhoto" class="profile-main-photo"></div>
                <h2 id="profileName" class="profile-name"></h2>
                <p id="profileLocation" class="profile-location"></p>
                <div class="profile-actions">
                    <button id="profilePageEditBtn" class="profile-action-btn edit hidden">Edit</button>
                    <button id="profileChatBtn" class="profile-action-btn chat">Chat</button>
                </div>
            </div>
            <div class="profile-section"><h3>About me</h3><p id="profileAboutMe"></p></div>
            <div class="profile-section"><h3>More info</h3><div class="info-grid" id="profileInfoGrid"></div></div>
            <div class="profile-section"><h3>Interests</h3><div class="interests-container" id="profileInterests"></div></div>
            <div class="profile-section"><h3>Pictures</h3><div class="profile-gallery" id="profilePictures"></div></div>
        </div>
    </div>
    
    <div id="editProfilePageSection" class="hidden">
        <div class="back-to-nearby" id="editProfileBackBtn">← Back to Main</div>
        <form id="editProfileForm" class="edit-profile-page">
            <div class="sidebar-profile" id="editSidebar"></div>
            <div class="profile-card">
                <div class="profile-section">
                    <div class="profile-section-header"><h3>Edit Info</h3></div>
                    <div class="info-grid">
                        <div class="form-group"><label>Name</label><input type="text" id="editNameInput"></div>
                        <div class="form-group"><label>Age</label><input type="number" id="editAgeInput"></div>
                        <div class="form-group"><label>Country</label><input type="text" id="editCountryInput"></div>
                        <div class="form-group"><label>Height (e.g., 1.78)</label><input type="text" id="editHeightInput"></div>
                        <div class="form-group"><label>Relationship Status</label><input type="text" id="editStatusInput"></div>
                        <div class="form-group"><label>Looking For</label><input type="text" id="editLookingForInput"></div>
                    </div>
                </div>
                <div class="profile-section">
                    <div class="profile-section-header"><h3>About Me</h3></div>
                    <textarea id="editAboutMeInput" rows="4"></textarea>
                </div>
                <div class="profile-section">
                    <div class="profile-section-header"><h3>Interests</h3></div>
                    <input type="text" id="editInterestsInput" placeholder="Enter interests, separated by commas">
                </div>
                <div class="profile-section">
                    <div class="profile-section-header"><h3>Pictures</h3></div>
                    <div class="photo-gallery" id="photoGallery"></div>
                     <input type="file" id="galleryUploadInput" class="hidden" accept="image/*" multiple>
                </div>
                <button type="submit" class="btn-submit" style="margin-top: 30px;">Save Changes</button>
            </div>
        </form>
    </div>
</div>

<div id="changePasswordModal" class="modal">
    <div class="modal-content">
        <button class="modal-close-btn">×</button>
        <h2>Change Password</h2>
        <form id="changePasswordForm" style="margin:0; padding:0; box-shadow:none; background:none;">
            <div class="form-group"><label for="newPasswordInput">New Password</label><input type="password" id="newPasswordInput" required minlength="6" autocomplete="new-password"></div>
            <button type="submit" class="btn-submit">Change Password</button>
        </form>
    </div>
</div>

<div id="chatModal" class="modal">
    <div class="chat-modal-content">
        <div class="chat-header">
            <div id="chatAvatar" class="chat-avatar"></div>
            <div class="chat-user-info"><div id="chatName" class="name"></div></div>
            <button id="chatCloseBtn" class="chat-close-btn">×</button>
        </div>
        <div id="chatMessagesArea" class="chat-messages-area"></div>
        <form id="chatForm" class="chat-form">
            <button type="button" id="uploadBtn" class="upload-btn">↑</button>
            <input type="file" id="fileInput" class="hidden" accept="image/*,video/*">
            <input type="text" id="chatMessageInput" placeholder="Сообщение..." autocomplete="off">
            <button type="submit" id="sendMessageBtn">➤</button>
        </form>
    </div>
</div>

<script>
    document.addEventListener('DOMContentLoaded', () => {
        const SUPABASE_URL = 'https://qufscnmganybuhrrjpvg.supabase.co';
        const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InF1ZnNjbm1nYW55YnVocnJqcHZnIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTQwMzEzMzQsImV4cCI6MjA2OTYwNzMzNH0.RlP6Bfafu5Fri_b6WUeXbsMyLbOfOIEhJRMpzttFsnE';
        const PHOTOS_BUCKET = 'photos';
        const CHAT_BUCKET = 'chatfiles';
        const { createClient } = supabase;
        const _supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
        
        let currentUser = null;
        let userMap = new Map();
        let viewedUser = null;
        let currentChatPartner = null;
        let messageSubscription = null;
        
        const mainNav = document.getElementById('mainNav');
        const loginNavBtn = document.getElementById('loginNavBtn');
        const logoutBtn = document.getElementById('logoutBtn');
        const profileSection = document.getElementById('profileSection');
        const registerSection = document.getElementById('registerSection');
        const loginSection = document.getElementById('loginSection');
        const usersGrid = document.getElementById('usersGrid');
        const changePasswordModal = document.getElementById('changePasswordModal');
        const changePasswordForm = document.getElementById('changePasswordForm');
        const loginForm = document.getElementById('loginForm');
        const registerForm = document.getElementById('registerForm');
        const showLoginFormLink = document.getElementById('showLoginFormLink');
        const showRegisterFormLink = document.getElementById('showRegisterFormLink');
        const editProfilePageSection = document.getElementById('editProfilePageSection');
        const editProfileForm = document.getElementById('editProfileForm');
        const editProfileBackBtn = document.getElementById('editProfileBackBtn');
        const photoGallery = document.getElementById('photoGallery');
        const galleryUploadInput = document.getElementById('galleryUploadInput');
        const viewProfilePageSection = document.getElementById('viewProfilePageSection');
        const viewProfileBackBtn = document.getElementById('viewProfileBackBtn');
        const profileChatBtn = document.getElementById('profileChatBtn');
        const profilePageEditBtn = document.getElementById('profilePageEditBtn');
        const chatModal = document.getElementById('chatModal');
        const chatForm = document.getElementById('chatForm');
        const uploadBtn = document.getElementById('uploadBtn');
        const fileInput = document.getElementById('fileInput');
        const chatCloseBtn = document.getElementById('chatCloseBtn');

        function showUI(view) {
            profileSection.classList.add('hidden');
            registerSection.classList.add('hidden');
            loginSection.classList.add('hidden');
            editProfilePageSection.classList.add('hidden');
            viewProfilePageSection.classList.add('hidden');
            const isMainView = ['profile', 'editProfilePage', 'viewProfilePage'].includes(view);
            mainNav.classList.toggle('hidden', !isMainView);
            logoutBtn.classList.toggle('hidden', !isMainView);
            loginNavBtn.classList.toggle('hidden', isMainView);
            const sectionToShow = document.getElementById(`${view}Section`);
            if (sectionToShow) sectionToShow.classList.remove('hidden');
        }

        function populateSidebar(containerId, user) {
            const sidebar = document.getElementById(containerId);
            if (!sidebar || !user) return;
            const isMainSidebar = containerId === 'mainSidebar';
            sidebar.innerHTML = `
                <div class="sidebar-avatar-container">
                  <div class="sidebar-avatar" style="background-image: url(${user.avatar_url || ''});">
                    ${!user.avatar_url ? (user.gender ? user.gender.charAt(0).toUpperCase() : 'U') : ''}
                  </div>
                  ${isMainSidebar ? `<div class="upload-avatar-overlay" id="avatarUploadTrigger">Сменить</div><input type="file" id="avatarUploadInput" class="hidden" accept="image/*">` : ''}
                </div>
                <div class="sidebar-name">${user.name || 'User'}, ${user.age || ''}</div>
                <div class="sidebar-location">${user.country || 'Location'}</div>
                ${isMainSidebar ? `
                <div class="sidebar-actions">
                    <button class="sidebar-btn primary" id="editProfileBtn">My Profile</button>
                    <button class="sidebar-btn secondary" id="changePasswordBtn">Change Password</button>
                </div>` : ''}
            `;
            if (isMainSidebar) {
                const editProfileBtn = document.getElementById('editProfileBtn');
                const changePasswordBtn = document.getElementById('changePasswordBtn');
                const avatarUploadTrigger = document.getElementById('avatarUploadTrigger');
                const avatarUploadInput = document.getElementById('avatarUploadInput');
                if(editProfileBtn) editProfileBtn.addEventListener('click', () => {
                    populateEditProfileForm();
                    showUI('editProfilePage');
                });
                if(changePasswordBtn) changePasswordBtn.addEventListener('click', () => changePasswordModal.classList.add('is-open'));
                if(avatarUploadTrigger) avatarUploadTrigger.addEventListener('click', () => avatarUploadInput.click());
                if(avatarUploadInput) avatarUploadInput.addEventListener('change', (e) => { if (e.target.files[0]) handlePhotoUpload([e.target.files[0]], true); });
            }
        }

        function populateEditProfileForm() {
            if (!currentUser) return;
            populateSidebar('editSidebar', currentUser);
            document.getElementById('editNameInput').value = currentUser.name || '';
            document.getElementById('editAgeInput').value = currentUser.age || '';
            document.getElementById('editCountryInput').value = currentUser.country || '';
            document.getElementById('editHeightInput').value = currentUser.height || '';
            document.getElementById('editStatusInput').value = currentUser.relationship_status || '';
            document.getElementById('editLookingForInput').value = currentUser.looking_for || '';
            document.getElementById('editAboutMeInput').value = currentUser.about_me || '';
            document.getElementById('editInterestsInput').value = (currentUser.interests || []).join(', ');
            renderPhotoGallery();
        }
        
        async function generateUserCards() {
            if (!usersGrid || !currentUser) return;
            usersGrid.innerHTML = '<p>Loading users...</p>';
            const { data: users, error } = await _supabase.from('users').select('*').neq('id', currentUser.id);
            if (error) {
                 usersGrid.innerHTML = `<p style="color:red">Error: ${error.message}</p>`;
                 return;
            }
            if (!users.length) {
                usersGrid.innerHTML = '<h3>No other users found</h3>';
                return;
            }
            userMap = new Map(users.map(user => [user.id, user]));
            usersGrid.innerHTML = users.map(user => {
                const name = user.name || 'User';
                const photoBg = user.avatar_url ? `url(${user.avatar_url})` : 'linear-gradient(135deg, #ff9ecf, #ffc3e8)';
                const photoContent = user.avatar_url ? '' : (user.name ? user.name.charAt(0).toUpperCase() : 'U');
                return `
                    <div class="user-card">
                        <div class="user-photo" data-user-id="${user.id}" style="background-image: ${photoBg};">${photoContent}</div>
                        <div class="user-info">
                            <div class="user-name">${user.name || 'User'}, ${user.age || ''}</div>
                            <div class="user-location">${user.country || 'Unknown'}</div>
                            <div class="user-actions">
                                <button class="action-btn-circle btn-message" title="Message" data-user-id="${user.id}">✉️</button>
                            </div>
                        </div>
                    </div>`;
            }).join('');
        }
        
        function showUserProfileView() {
            if (!currentUser) return;
            populateSidebar('mainSidebar', currentUser);
            generateUserCards();
            showUI('profile');
        }
        
        function renderFullProfile(user, isMyProfile = false) {
            viewedUser = user;
            const photoDiv = document.getElementById('profileMainPhoto');
            if (user.avatar_url) {
                photoDiv.style.backgroundImage = `url(${user.avatar_url})`;
                photoDiv.textContent = '';
            } else {
                photoDiv.style.backgroundImage = 'linear-gradient(135deg, #a1c4fd, #c2e9fb)';
                photoDiv.textContent = user.name ? user.name.charAt(0).toUpperCase() : 'U';
            }
            document.getElementById('profileName').textContent = `${user.name || 'User'}, ${user.age || ''}`;
            document.getElementById('profileLocation').textContent = user.country || 'Unknown';
            document.getElementById('profileAboutMe').textContent = user.about_me || 'No information provided.';
            const infoGrid = document.getElementById('profileInfoGrid');
            infoGrid.innerHTML = '';
            const info = { Height: user.height, Status: user.relationship_status, "Looking For": user.looking_for };
            for (const [key, value] of Object.entries(info)) {
                if (value) infoGrid.innerHTML += `<div class="info-item"><span class="info-label">${key}</span><span class="info-value">${value}</span></div>`;
            }
            const interestsContainer = document.getElementById('profileInterests');
            interestsContainer.innerHTML = '';
            if (user.interests && user.interests.length) {
                user.interests.forEach(interest => { interestsContainer.innerHTML += `<div class="interest-tag">${interest}</div>`; });
            }
            const picturesContainer = document.getElementById('profilePictures');
            picturesContainer.innerHTML = '';
            if (user.photos_urls && user.photos_urls.length) {
                user.photos_urls.forEach(photo => { picturesContainer.innerHTML += `<img src="${photo.url}" class="profile-gallery-img">`; });
            }
            profilePageEditBtn.classList.toggle('hidden', !isMyProfile);
            profileChatBtn.classList.toggle('hidden', isMyProfile);
        }

        async function showUserProfile(userId, isMyProfile = false) {
            const { data: user, error } = await _supabase.from('users').select('*').eq('id', userId).single();
            if (error || !user) return alert("Could not load profile.");
            renderFullProfile(user, isMyProfile);
            showUI('viewProfilePage');
        }
    
        function renderPhotoGallery() {
            if (!photoGallery || !currentUser) return;
            photoGallery.innerHTML = '';
            const photos = currentUser.photos_urls || [];
            photos.forEach(photo => {
                const item = document.createElement('div');
                item.className = 'gallery-item';
                item.innerHTML = `
                    <img src="${photo.url}" class="gallery-img">
                    <button class="delete-photo-btn" data-url="${photo.url}">×</button>
                    ${photo.is_main ? '<div class="main-photo-check">✔</div>' : ''}
                `;
                const img = item.querySelector('.gallery-img');
                if(img && !photo.is_main) {
                    img.addEventListener('click', () => {
                        if (confirm('Сделать это фото главным?')) setMainPhoto(photo.url);
                    });
                }
                photoGallery.appendChild(item);
            });
            const addCard = document.createElement('div');
            addCard.className = 'add-photo-card';
            addCard.innerHTML = `+<span>Add</span>`;
            addCard.onclick = () => galleryUploadInput.click();
            photoGallery.appendChild(addCard);
        }

        async function handlePhotoUpload(files, isAvatar) {
            if (!files.length) return;
            let photos = currentUser.photos_urls || [];
            let newAvatarUrl = currentUser.avatar_url;
            for (const file of files) {
                const filePath = `${currentUser.id}/${Date.now()}-${file.name}`;
                const { error } = await _supabase.storage.from(PHOTOS_BUCKET).upload(filePath, file);
                if (error) { alert(`Ошибка загрузки файла: ${file.name}`); continue; }
                const { data: { publicUrl } } = _supabase.storage.from(PHOTOS_BUCKET).getPublicUrl(filePath);
                const isMain = isAvatar || photos.length === 0;
                photos.push({ url: publicUrl, is_main: isMain });
                if (isMain) newAvatarUrl = publicUrl;
            }
            if (isAvatar) {
                photos = photos.map(p => ({ ...p, is_main: p.url === newAvatarUrl }));
            }
            const { data, error } = await _supabase.from('users').update({ photos_urls: photos, avatar_url: newAvatarUrl }).eq('id', currentUser.id).select().single();
            if (error) return alert('Ошибка обновления профиля.');
            currentUser = data;
            populateEditProfileForm();
        }
        
        async function setMainPhoto(selectedUrl) {
            const updatedPhotos = (currentUser.photos_urls || []).map(p => ({ ...p, is_main: p.url === selectedUrl }));
            const { data, error } = await _supabase.from('users').update({ photos_urls: updatedPhotos, avatar_url: selectedUrl }).eq('id', currentUser.id).select().single();
            if (error) return alert('Ошибка обновления главного фото.');
            currentUser = data;
            populateEditProfileForm();
        }
        
        async function handleDeletePhoto(urlToDelete) {
            let photos = (currentUser.photos_urls || []).filter(p => p.url !== urlToDelete);
            let newAvatarUrl = currentUser.avatar_url;
            const deletedPhotoWasMain = currentUser.avatar_url === urlToDelete;
            if (deletedPhotoWasMain && photos.length > 0) {
                photos[0].is_main = true;
                newAvatarUrl = photos[0].url;
            } else if (photos.length === 0) { newAvatarUrl = null; }
            const { data, error } = await _supabase.from('users').update({ photos_urls: photos, avatar_url: newAvatarUrl }).eq('id', currentUser.id).select().single();
            if (error) return alert('Ошибка удаления фото.');
            const path = new URL(urlToDelete).pathname.split(`/${PHOTOS_BUCKET}/`)[1];
            await _supabase.storage.from(PHOTOS_BUCKET).remove([path]);
            currentUser = data;
            populateEditProfileForm();
        }

        async function openChat(partnerUser) {
            if(!partnerUser) return;
            currentChatPartner = partnerUser;
            const chatName = document.getElementById('chatName');
            const chatAvatar = document.getElementById('chatAvatar');
            if (chatName) chatName.textContent = partnerUser.name || 'User';
            if (chatAvatar) {
                if(partnerUser.avatar_url) {
                    chatAvatar.style.backgroundImage = `url(${partnerUser.avatar_url})`;
                    chatAvatar.textContent = '';
                } else {
                    chatAvatar.style.backgroundImage = '';
                    chatAvatar.textContent = partnerUser.name ? partnerUser.name.charAt(0).toUpperCase() : 'U';
                }
            }
            chatModal.classList.add('is-open');
            await fetchMessages(partnerUser.id);
            listenForNewMessages(partnerUser.id);
        }
        
        function closeChat() {
            chatModal.classList.remove('is-open');
            if(messageSubscription) {
                _supabase.removeChannel(messageSubscription);
                messageSubscription = null;
            }
        }

        function renderMessage(message) {
            const messagesArea = document.getElementById('chatMessagesArea');
            if(!messagesArea) return;
            const bubble = document.createElement('div');
            bubble.classList.add('message-bubble', message.sender_id === currentUser.id ? 'sent' : 'received');
            if (message.message_type === 'text' && message.content) {
                bubble.textContent = message.content;
            } else if (message.attachments) {
                bubble.classList.add('has-attachment');
                let mediaElement;
                if (message.message_type === 'image') mediaElement = document.createElement('img');
                else if (message.message_type === 'video') mediaElement = document.createElement('video');
                if (mediaElement) {
                    mediaElement.src = message.attachments;
                    mediaElement.classList.add('message-attachment');
                    if (message.message_type === 'video') mediaElement.controls = true;
                    bubble.appendChild(mediaElement);
                }
            }
            messagesArea.appendChild(bubble);
            messagesArea.scrollTop = messagesArea.scrollHeight;
        }

        async function fetchMessages(partnerId) {
            const messagesArea = document.getElementById('chatMessagesArea');
            if(!messagesArea) return;
            messagesArea.innerHTML = '';
            const orFilter = `and(sender_id.eq.${currentUser.id},receiver_id.eq.${partnerId}),and(sender_id.eq.${partnerId},receiver_id.eq.${currentUser.id})`;
            const { data, error } = await _supabase.from('messages').select('*').or(orFilter).order('created_at', { ascending: true });
            if (error) console.error('Error fetching messages:', error);
            else data.forEach(renderMessage);
        }

        function listenForNewMessages(partnerId) {
            if (messageSubscription) _supabase.removeChannel(messageSubscription);
            messageSubscription = _supabase.channel('public:messages')
                .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, 
                (payload) => {
                    if (currentChatPartner && ((payload.new.receiver_id === currentUser.id && payload.new.sender_id === partnerId) || 
                        (payload.new.sender_id === currentUser.id && payload.new.receiver_id === partnerId))) {
                       // We don't need to render our own messages here because of optimistic rendering
                       if (payload.new.sender_id !== currentUser.id) {
                           renderMessage(payload.new);
                       }
                    }
                }).subscribe();
        }
        
        async function handleChatFileUpload(file) {
            if (!file || !currentChatPartner) return;
            uploadBtn.disabled = true;
            const filePath = `${currentUser.id}/${Date.now()}-${file.name}`;
            const { error: uploadError } = await _supabase.storage.from(CHAT_BUCKET).upload(filePath, file);
            uploadBtn.disabled = false;
            if (uploadError) {
                console.error("Upload error:", uploadError);
                return alert("Не удалось загрузить файл.");
            }
            const { data: urlData } = _supabase.storage.from(CHAT_BUCKET).getPublicUrl(filePath);
            const messageType = file.type.startsWith('image/') ? 'image' : 'video';
            const messagePayload = { sender_id: currentUser.id, receiver_id: currentChatPartner.id, content: '', message_type: messageType, attachments: urlData.publicUrl, created_at: new Date().toISOString() };
            
            renderMessage(messagePayload); // Optimistic render

            const { error: insertError } = await _supabase.from('messages').insert({ sender_id: messagePayload.sender_id, receiver_id: messagePayload.receiver_id, content: messagePayload.content, message_type: messageType, attachments: messagePayload.attachments });
            if (insertError) {
                console.error("Error sending file message:", insertError);
                alert("Не удалось отправить файл.");
            }
        }

        async function init() {
            if (loginNavBtn) loginNavBtn.addEventListener('click', () => showUI('login'));
            if (logoutBtn) logoutBtn.addEventListener('click', async () => { await _supabase.auth.signOut(); currentUser = null; showUI('register'); });
            if (showLoginFormLink) showLoginFormLink.addEventListener('click', (e) => { e.preventDefault(); showUI('login'); });
            if (showRegisterFormLink) showRegisterFormLink.addEventListener('click', (e) => { e.preventDefault(); showUI('register'); });
            if (editProfileBackBtn) editProfileBackBtn.addEventListener('click', () => showUserProfileView());
            if (viewProfileBackBtn) viewProfileBackBtn.addEventListener('click', () => showUserProfileView());

            if (loginForm) loginForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const { data, error } = await _supabase.auth.signInWithPassword({ email: document.getElementById('loginEmail').value, password: document.getElementById('loginPassword').value });
                if (error) return alert(error.message);
                const { data: profile } = await _supabase.from('users').select('*').eq('id', data.user.id).single();
                currentUser = profile;
                showUserProfileView();
            });

            if (registerForm) registerForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const { data, error } = await _supabase.auth.signUp({ email: document.getElementById('registerEmail').value, password: document.getElementById('registerPassword').value });
                if (error) return alert(error.message);
                const { error: profileError } = await _supabase.from('users').update({ name: document.getElementById('registerName').value, age: document.getElementById('registerAge').value, country: document.getElementById('registerCountry').value, gender: document.getElementById('registerGender').value }).eq('id', data.user.id);
                if (profileError) return alert(profileError.message);
                const { data: profile } = await _supabase.from('users').select('*').eq('id', data.user.id).single();
                currentUser = profile;
                showUserProfileView();
            });
            
            if (editProfileForm) editProfileForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const updates = {
                    name: document.getElementById('editNameInput').value,
                    age: document.getElementById('editAgeInput').value,
                    country: document.getElementById('editCountryInput').value,
                    height: document.getElementById('editHeightInput').value,
                    relationship_status: document.getElementById('editStatusInput').value,
                    looking_for: document.getElementById('editLookingForInput').value,
                    about_me: document.getElementById('editAboutMeInput').value,
                    interests: document.getElementById('editInterestsInput').value.split(',').map(item => item.trim()).filter(Boolean),
                };
                const { data, error } = await _supabase.from('users').update(updates).eq('id', currentUser.id).select().single();
                if (error) return alert(error.message);
                currentUser = data;
                alert('Profile saved!');
                showUserProfileView();
            });

            if (changePasswordForm) changePasswordForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const { error } = await _supabase.auth.updateUser({ password: document.getElementById('newPasswordInput').value });
                if (error) return alert(error.message);
                document.getElementById('newPasswordInput').value = '';
                changePasswordModal.classList.remove('is-open');
                alert('Password updated!');
            });
            
            if (usersGrid) usersGrid.addEventListener('click', (e) => {
                const messageButton = e.target.closest('.btn-message');
                if (messageButton) {
                    e.stopPropagation();
                    const partnerId = messageButton.dataset.userId;
                    const partnerUser = userMap.get(partnerId);
                    if (partnerUser) openChat(partnerUser);
                } else {
                    const cardPhoto = e.target.closest('.user-photo');
                    if(cardPhoto) {
                        const card = cardPhoto.closest('.user-card');
                        if (card && card.dataset.userId) {
                            showUserProfile(card.dataset.userId, false);
                        }
                    }
                }
            });
            
            if (profileChatBtn) profileChatBtn.addEventListener('click', () => {
                if (viewedUser) openChat(viewedUser);
            });

            if (chatForm) chatForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const messageInput = document.getElementById('chatMessageInput');
                const content = messageInput.value.trim();
                if (content && currentChatPartner) {
                    const messagePayload = { sender_id: currentUser.id, receiver_id: currentChatPartner.id, content: content, message_type: 'text', created_at: new Date().toISOString() };
                    renderMessage(messagePayload);
                    messageInput.value = '';
                    const { error } = await _supabase.from('messages').insert({ sender_id: messagePayload.sender_id, receiver_id: messagePayload.receiver_id, content: messagePayload.content, message_type: 'text' });
                    if (error) { 
                        console.error('Error sending message:', error);
                        alert("Сообщение не удалось отправить.");
                    }
                }
            });
            
            if(uploadBtn) uploadBtn.addEventListener('click', () => fileInput.click());
            if(fileInput) fileInput.addEventListener('change', (e) => { if (e.target.files.length > 0) handleChatFileUpload(e.target.files[0]); });
            if(chatCloseBtn) chatCloseBtn.addEventListener('click', closeChat);
            
            if (galleryUploadInput) galleryUploadInput.addEventListener('change', (e) => { if (e.target.files.length) handlePhotoUpload(Array.from(e.target.files), false); });
            
            if (photoGallery) photoGallery.addEventListener('click', (e) => {
                if (e.target.classList.contains('delete-photo-btn')) {
                    const urlToDelete = e.target.dataset.url;
                    if (confirm('Удалить это фото?')) handleDeletePhoto(urlToDelete);
                }
            });

            document.querySelectorAll('.modal-close-btn').forEach(btn => {
                btn.addEventListener('click', () => btn.closest('.modal').classList.remove('is-open'));
            });

            const { data: { session } } = await _supabase.auth.getSession();
            if (session?.user) {
                const { data: profile } = await _supabase.from('users').select('*').eq('id', session.user.id).single();
                if (profile) {
                    currentUser = profile;
                    showUserProfileView();
                } else { showUI('register'); }
            } else {
                showUI('register');
            }
        }
        
        init();
    });
</script>
</body>
</html>
