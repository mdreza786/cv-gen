# cv-gen
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive CV Builder - Secure Access</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap" rel="stylesheet">
    <style>
        /* Custom Tailwind Configuration for the app */
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        'primary': '#10b981', /* Emerald */
                        'secondary': '#1f2937', /* Gray-800 */
                        'background-light': '#f9fafb', /* Gray-50 */
                        'background-dark': '#111827', /* Gray-900 */
                        'accent-dark': '#374151', /* Gray-700 for input background */
                    }
                }
            }
        }

        /* Styles for Print View (CV should look clean when printed) */
        @media print {
            body { background-color: white !important; color: black !important; }
            #input-panel, #app-actions { display: none; } /* Hide input panel and actions on print */
            #cv-preview {
                width: 100% !important; max-width: none !important; margin: 0 !important; padding: 0 !important;
                box-shadow: none !important; border: none !important;
            }
            .cv-section h2 { color: #1f2937 !important; border-bottom-color: #d1d5db !important; }
            #cv-preview * { color: #111827 !important; background: none !important; }
        }

        /* CV Thumbnails Background for Login Page (No change needed here) */
        #login-wrapper {
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 100 100"><rect x="10" y="10" width="80" height="15" fill="%23d1d5db" /><rect x="10" y="30" width="50" height="5" fill="%239ca3af" /><rect x="10" y="40" width="80" height="40" fill="%23f3f4f6" /><rect x="15" y="45" width="20" height="5" fill="%23e5e7eb" /><rect x="15" y="55" width="70" height="3" fill="%23e5e7eb" /><rect x="15" y="62" width="50" height="3" fill="%23e5e7eb" /><rect x="15" y="69" width="70" height="3" fill="%23e5e7eb" /></svg>');
            background-size: 200px;
            opacity: 0.1; 
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 0;
        }
        
        /* Custom Input field style for consistency and better look */
        .input-field {
            width: 100%; padding: 0.6rem 0.75rem; border: 1px solid #4b5563; /* Gray-600 */
            background-color: #374151; /* Gray-700 (accent-dark) */
            border-radius: 0.5rem; color: #f9fafb; /* Gray-50 */
            transition: all 0.2s;
            box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.3); /* Subtle inner shadow for depth */
        }
        .input-field:focus {
            outline: none; box-shadow: 0 0 0 3px rgba(16, 185, 129, 0.5); /* Primary color focus ring */
            border-color: #10b981;
        }

        /* Styling for the large OTP input */
        #otp-code {
            font-size: 1.5rem; /* Larger font */
            letter-spacing: 0.5rem; /* Spacing for digits */
            text-align: center;
            padding: 1rem;
        }
    </style>
</head>
<body class="bg-background-dark text-gray-100 font-sans antialiased">
    
    <div id="login-container" class="min-h-screen flex items-center justify-center p-4 relative">
        <div id="login-wrapper"></div> <div class="w-full max-w-md bg-secondary p-8 rounded-xl shadow-2xl border border-gray-700 z-10">
            <h2 class="text-3xl font-bold mb-8 text-primary text-center">CV Generator Access</h2>
            
            <form id="mobile-register-form" class="space-y-6" onsubmit="event.preventDefault(); handleMobileRegister()">
                <h3 class="text-xl text-white font-semibold mb-4 border-b border-gray-700 pb-2">Enter Mobile</h3>
                <div>
                    <label for="mobile-number" class="block mb-2 text-sm font-medium text-gray-400">Mobile Number</label>
                    <input type="tel" id="mobile-number" placeholder="+1 555 123 4567" required class="input-field">
                </div>
                <button type="submit" class="w-full bg-gray-600 text-white py-3 font-bold rounded-lg hover:bg-gray-500 transition duration-200 shadow-lg shadow-gray-700/50">
                    Send OTP
                </button>
                <p id="auth-message-mobile" class="text-center text-sm font-medium text-red-400"></p>
            </form>

            <form id="otp-verification-form" class="space-y-6 hidden" onsubmit="event.preventDefault(); handleOTPVerification()">
                <h3 class="text-xl text-white font-semibold mb-4 border-b border-gray-700 pb-2">Verify OTP</h3>
                <div class="p-3 bg-gray-700 rounded-lg text-center font-mono text-lg border border-primary/50">
                    <span class="text-yellow-400 font-medium">Simulated OTP:</span> <span class="text-primary font-bold">123456</span>
                </div>
                <div>
                    <label for="otp-code" class="block mb-2 text-sm font-medium text-gray-400">Enter 6-Digit Code</label>
                    <input type="text" id="otp-code" placeholder="------" required class="input-field" maxlength="6">
                </div>
                <button type="submit" class="w-full bg-primary text-white py-3 font-bold rounded-lg hover:bg-emerald-500 transition duration-200 shadow-lg shadow-primary/50">
                    Verify & Log In
                </button>
                <button type="button" onclick="showMobileRegister()" class="w-full text-sm text-gray-400 py-2 hover:text-white transition duration-200">
                    Change Number / Resend
                </button>
                <p id="auth-message-otp" class="text-center text-sm font-medium text-red-400"></p>
            </form>

            <p class="text-center text-xs mt-6 text-gray-500">
                Secure access powered by simulated verification.
            </p>
        </div>
    </div>

    <div id="main-cv-app" class="hidden min-h-screen flex flex-col lg:flex-row">
        
        <div id="input-panel" class="lg:w-1/3 p-6 bg-secondary text-gray-200 shadow-2xl shadow-gray-900/50 overflow-y-auto max-h-screen">
            <h1 class="text-3xl font-black mb-6 text-primary border-b-4 border-primary/50 pb-2">CV Builder 🚀</h1>
            <p class="text-sm mb-6 text-gray-400">Your professional resume updates instantly on the right.</p>
            
            <div id="app-actions" class="mb-8 space-y-3">
                <button type="button" onclick="window.print()" class="w-full bg-green-700 text-white py-3 font-bold rounded-xl hover:bg-green-600 transition duration-200 shadow-xl shadow-green-900/50 flex items-center justify-center space-x-2">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                    <span>Download / Save as PDF</span>
                </button>
                <button type="button" onclick="handleLogout()" class="w-full text-sm text-red-400 py-2 font-medium hover:text-red-300 transition duration-200 flex items-center justify-center space-x-1">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H7a3 3 0 01-3-3v-5m-4 0v-5a3 3 0 013-3h5m0 0l-4-4m0 0l-4 4"></path></svg>
                    <span>Logout</span>
                </button>
            </div>
            
            <form id="cv-form" class="space-y-6">
                
                <div class="space-y-2 p-4 bg-gray-800/50 rounded-lg border border-gray-700">
                    <h2 class="text-xl font-semibold text-white">1. Personal Details</h2>
                    <input type="text" id="name" placeholder="Full Name (e.g., Jane Doe)" class="input-field" oninput="generateCV()">
                    <input type="text" id="title" placeholder="Professional Title (e.g., Web Developer)" class="input-field" oninput="generateCV()">
                    <input type="email" id="email" placeholder="Email Address" class="input-field" oninput="generateCV()">
                    <input type="tel" id="phone" placeholder="Phone Number" class="input-field" oninput="generateCV()">
                    <input type="text" id="location" placeholder="Location (e.g., London, UK)" class="input-field" oninput="generateCV()">
                </div>
                <div class="space-y-2 p-4 bg-gray-800/50 rounded-lg border border-gray-700">
                    <h2 class="text-xl font-semibold text-white">2. Professional Summary</h2>
                    <textarea id="summary" placeholder="A brief, powerful summary of your career and goals." class="input-field h-24" oninput="generateCV()"></textarea>
                </div>
                <div class="space-y-2 p-4 bg-gray-800/50 rounded-lg border border-gray-700">
                    <h2 class="text-xl font-semibold text-white">3. Core Skills</h2>
                    <textarea id="skills" placeholder="HTML, CSS, JavaScript, etc. (Separate with commas)" class="input-field h-20" oninput="generateCV()">HTML, CSS, JavaScript, Tailwind CSS, Project Management</textarea>
                </div>
                <div class="space-y-2 p-4 bg-gray-800/50 rounded-lg border border-gray-700">
                    <h2 class="text-xl font-semibold text-white">4. Experience (Job 1)</h2>
                    <input type="text" id="jobTitle1" placeholder="Job Title" class="input-field" oninput="generateCV()">
                    <input type="text" id="company1" placeholder="Company Name" class="input-field" oninput="generateCV()">
                    <input type="text" id="duration1" placeholder="Duration (e.g., 2020 - Present)" class="input-field" oninput="generateCV()">
                    <textarea id="responsibilities1" placeholder="Key responsibilities/achievements (Separate with commas)" class="input-field h-20" oninput="generateCV()"></textarea>
                </div>
                
                <div class="space-y-2 p-4 bg-gray-800/50 rounded-lg border border-gray-700">
                    <h2 class="text-xl font-semibold text-white">5. Education</h2>
                    <input type="text" id="degree" placeholder="Degree/Certificate" class="input-field" oninput="generateCV()">
                    <input type="text" id="institution" placeholder="Institution Name" class="input-field" oninput="generateCV()">
                    <input type="text" id="eduDuration" placeholder="Years Attended" class="input-field" oninput="generateCV()">
                </div>
            </form>
        </div>
        
        <div class="lg:w-2/3 p-4 lg:p-10 bg-gray-700 flex justify-center items-start min-h-screen">
            <div id="cv-preview" class="w-full max-w-3xl bg-white text-gray-900 shadow-2xl p-8 md:p-12 lg:p-16 rounded-lg border border-gray-200">
                </div>
        </div>
    </div>
    
    <script>
        // --- AUTHENTICATION FUNCTIONS (Simulated with Mobile/OTP) ---
        const USER_MOBILE_KEY = 'cv_builder_mobile';
        const AUTH_TOKEN_KEY = 'cv_builder_auth';
        const SIMULATED_OTP = '123456'; 

        function checkAuth() {
            const token = localStorage.getItem(AUTH_TOKEN_KEY);
            
            if (token) {
                document.getElementById('login-container').classList.add('hidden');
                document.getElementById('main-cv-app').classList.remove('hidden');
                initializeCVBuilder();
            } else {
                document.getElementById('login-container').classList.remove('hidden');
                document.getElementById('main-cv-app').classList.add('hidden');
                showMobileRegister(); 
            }
        }

        function showMobileRegister() {
            document.getElementById('mobile-register-form').classList.remove('hidden');
            document.getElementById('otp-verification-form').classList.add('hidden');
            document.getElementById('auth-message-mobile').textContent = '';
            document.getElementById('auth-message-otp').textContent = '';
        }

        function handleMobileRegister() {
            const mobileNumber = document.getElementById('mobile-number').value.trim();
            const msg = document.getElementById('auth-message-mobile');

            if (mobileNumber.length < 10) {
                msg.textContent = 'Please enter a valid mobile number.';
                return;
            }

            localStorage.setItem(USER_MOBILE_KEY, mobileNumber);
            msg.textContent = `OTP sent to ${mobileNumber}. Check the hint above.`;
            msg.classList.add('text-primary');
            msg.classList.remove('text-red-400');

            setTimeout(() => {
                document.getElementById('mobile-register-form').classList.add('hidden');
                document.getElementById('otp-verification-form').classList.remove('hidden');
                document.getElementById('auth-message-mobile').textContent = '';
                document.getElementById('otp-code').focus(); // Focus the OTP input
            }, 1500);
        }

        function handleOTPVerification() {
            const otpInput = document.getElementById('otp-code').value.trim();
            const msg = document.getElementById('auth-message-otp');

            if (otpInput === SIMULATED_OTP) {
                const mobile = localStorage.getItem(USER_MOBILE_KEY);
                localStorage.setItem(AUTH_TOKEN_KEY, `token_for_${mobile}_${Date.now()}`);
                
                msg.textContent = 'Verification successful! Logging in...';
                msg.classList.add('text-primary');
                msg.classList.remove('text-red-400');

                setTimeout(checkAuth, 500); 

            } else {
                msg.textContent = 'Invalid OTP. Please try again.';
                msg.classList.add('text-red-400');
                msg.classList.remove('text-primary');
            }
        }

        function handleLogout() {
            localStorage.removeItem(AUTH_TOKEN_KEY);
            document.getElementById('cv-form').reset();
            checkAuth();
        }

        // --- CV BUILDER INITIALIZATION ---
        function initializeCVBuilder() {
            if (document.getElementById('name').value !== "") return;
            
            document.getElementById('name').value = "Johnathan R. Smith";
            document.getElementById('title').value = "Full Stack Web Developer";
            document.getElementById('email').value = "john.smith@example.com";
            document.getElementById('phone').value = "+1 (555) 123-4567";
            document.getElementById('location').value = "San Francisco, CA";
            document.getElementById('summary').value = "Highly motivated and results-driven Web Developer with 5 years of experience building and deploying responsive, high-performance web applications. Expertise in the MERN stack, focused on clean code architecture and seamless user experiences.";
            document.getElementById('jobTitle1').value = "Senior Front-end Developer";
            document.getElementById('company1').value = "Tech Solutions Inc.";
            document.getElementById('duration1').value = "Jan 2022 - Present";
            document.getElementById('responsibilities1').value = "Led a team of 3 developers in refactoring legacy code into a modern React architecture, resulting in a 40% performance increase, Designed and implemented responsive UI components using Tailwind CSS and vanilla JavaScript, improving mobile conversion rates by 15%";
            document.getElementById('degree').value = "B.S. in Computer Science";
            document.getElementById('institution').value = "University of Technology";
            document.getElementById('eduDuration').value = "2018 - 2022";
            
            generateCV();
        }

        // --- CV GENERATION ---
        function generateCV() {
            const data = {
                name: document.getElementById('name').value.trim(),
                title: document.getElementById('title').value.trim(),
                email: document.getElementById('email').value.trim(),
                phone: document.getElementById('phone').value.trim(),
                location: document.getElementById('location').value.trim(),
                summary: document.getElementById('summary').value.trim(),
                skills: document.getElementById('skills').value.trim().split(',').map(s => s.trim()).filter(s => s),
                
                experience: [{
                    title: document.getElementById('jobTitle1').value.trim(),
                    company: document.getElementById('company1').value.trim(),
                    duration: document.getElementById('duration1').value.trim(),
                    responsibilities: document.getElementById('responsibilities1').value.trim().split(',').map(r => r.trim()).filter(r => r)
                }],
                
                education: {
                    degree: document.getElementById('degree').value.trim(),
                    institution: document.getElementById('institution').value.trim(),
                    duration: document.getElementById('eduDuration').value.trim()
                }
            };
            const preview = document.getElementById('cv-preview');
            let html = '';
            
            // 1. Header Section
            if (data.name) {
                const contactInfo = [
                    data.phone ? `<span class="whitespace-nowrap">${data.phone}</span>` : '',
                    data.email ? `<span class="whitespace-nowrap">${data.email}</span>` : '',
                    data.location ? `<span class="whitespace-nowrap">${data.location}</span>` : ''
                ].filter(Boolean).join('<span class="text-gray-400 font-normal mx-2">|</span>');
                
                html += `
                    <header class="text-center pb-6 border-b border-gray-300 mb-6">
                        <h1 class="text-4xl font-extrabold text-secondary">${data.name}</h1>
                        <p class="text-xl font-semibold text-primary mt-1">${data.title}</p>
                        <div class="mt-3 text-sm text-gray-600 flex justify-center items-center flex-wrap space-x-1">
                            ${contactInfo}
                        </div>
                    </header>
                `;
            }
            // 2. Summary Section
            if (data.summary) {
                html += `
                    <div class="cv-section mb-6">
                        <h2 class="text-2xl font-bold text-secondary border-b-2 border-primary pb-1 mb-3">Summary</h2>
                        <p class="text-gray-700">${data.summary}</p>
                    </div>
                `;
            }
            // 3. Skills Section (Added hover effect to tags)
            if (data.skills.length > 0) {
                html += `
                    <div class="cv-section mb-6">
                        <h2 class="text-2xl font-bold text-secondary border-b-2 border-primary pb-1 mb-3">Core Skills</h2>
                        <div class="flex flex-wrap gap-2 text-sm">
                            ${data.skills.map(skill => `<span class="bg-gray-100 text-gray-700 px-3 py-1 rounded-full font-medium border border-gray-300 transition duration-150 hover:bg-primary/10 hover:border-primary">${skill}</span>`).join('')}
                        </div>
                    </div>
                `;
            }
            // 4. Experience Section
            if (data.experience.some(exp => exp.title)) {
                html += `
                    <div class="cv-section mb-6">
                        <h2 class="text-2xl font-bold text-secondary border-b-2 border-primary pb-1 mb-3">Experience</h2>
                        ${data.experience.map(exp => `
                            <div class="mb-4">
                                <div class="flex justify-between items-start">
                                    <div>
                                        <h3 class="text-lg font-bold">${exp.title}</h3>
                                        <p class="text-gray-700 italic">${exp.company}</p>
                                    </div>
                                    <span class="text-sm font-semibold text-gray-600">${exp.duration}</span>
                                </div>
                                ${exp.responsibilities.length > 0 ? `
                                    <ul class="list-disc pl-5 mt-1 text-gray-700 text-sm">
                                        ${exp.responsibilities.map(r => `<li>${r}</li>`).join('')}
                                    </ul>
                                ` : ''}
                            </div>
                        `).join('')}
                    </div>
                `;
            }
            // 5. Education Section
            if (data.education.degree) {
                html += `
                    <div class="cv-section">
                        <h2 class="text-2xl font-bold text-secondary border-b-2 border-primary pb-1 mb-3">Education</h2>
                        <div class="flex justify-between items-start">
                            <div>
                                <h3 class="text-lg font-bold">${data.education.degree}</h3>
                                <p class="text-gray-700 italic">${data.education.institution}</p>
                            </div>
                            <span class="text-sm font-semibold text-gray-600">${data.education.duration}</span>
                        </div>
                    </div>
                `;
            }
            preview.innerHTML = html || '<p class="text-gray-500 italic text-center">Start typing in the Builder Panel to see your CV appear here...</p>';
        }
        
        // --- INITIALIZATION ---
        document.addEventListener('DOMContentLoaded', () => {
            checkAuth();
        });

    </script>
</body>
</html>
