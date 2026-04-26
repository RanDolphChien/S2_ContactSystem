# ContactSystem (聯絡單管理系統)

A modern web application for managing and printing class contact forms in Traditional Chinese. Designed for educators to create, edit, and distribute weekly class newsletters efficiently.

## Features

### 📋 **Core Functionality**
- **Single-Page Application**: No build system required - just open and use
- **Cloud-Synced Drafts**: Automatic save to Supabase with offline fallback to localStorage
- **Email/Password Authentication**: Secure user login with password recovery
- **Real-Time Content Editing**: Edit content and see changes instantly
- **Concurrent Login Prevention**: Prevents the same account from being logged in simultaneously
- **Session Management**: Automatic logout when switching accounts with user warnings

### 📄 **Print Features**
- **A4 Landscape Printing**: Optimized layout for efficient paper use
- **Two Pages Per Sheet**: Prints two half-pages side-by-side for 10 total printable pages
- **Dashed Cut Lines**: Visual guides for cutting pages
- **Print Preview**: WYSIWYG editing before printing
- **Professional Formatting**: Maintains borders, spacing, and layout on print

### ✍️ **Content Management**
- **5 Complete Forms**: 10 half-pages total (pages 1-10)
- **20 Vocabulary Items Per Form**: Editable vocab fields with copy button
- **Structured Sections Per Page**:
  - Title and date
  - Lesson information
  - Homework assignments
  - Communication notes
  - Vocabulary grid
- **Auto-Synchronization**: Left-page content automatically mirrors to right pages
- **Content Persistence**: Saves to cloud and local storage

### 🎨 **User Interface**
- **Sticky Control Bar**: Always-visible Print, Edit, and Logout buttons
- **Edit Mode Toggle**: Seamless switching between viewing and editing
- **Status Notifications**: Real-time feedback on sync status
- **Dark Print UI**: Optimized for screen readability
- **Responsive Design**: Built with Tailwind CSS for all screen sizes
- **Chinese Localization**: Full Traditional Chinese UI and support for Chinese text

### 🔒 **Security & Reliability**
- **User Authentication**: Email/password login via Supabase
- **Row-Level Security**: User data isolation
- **Password Recovery**: Built-in reset flow
- **Data Validation**: Automatic UPSERT prevents duplicate key errors
- **Error Handling**: Graceful fallback to localStorage on sync failures

## Tech Stack

- **Frontend**: Vanilla JavaScript + HTML + CSS
- **Styling**: Tailwind CSS + Google Fonts (Noto Sans TC)
- **Backend**: Supabase (PostgreSQL + Auth)
- **Storage**: Cloud (Supabase) + Local (Browser localStorage)
- **Hosting**: Static file hosting (works on any static server)

## Getting Started

### Prerequisites
- Modern web browser (Chrome, Firefox, Safari, Edge)
- Supabase project with:
  - Authentication enabled
  - `contact_drafts` table created
  - Users registered with email/password

### Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/RanDolphChien/ContactSystem.git
   cd ContactSystem
   ```

2. **Configure Supabase Credentials**
   - Open `ContactSystem.html` in a text editor
   - Locate lines 851-854 with Supabase credentials
   - Update with your Supabase project URL and anon key

3. **Open in Browser**
   - Simply open `index.html` in your web browser
   - Or serve with a local server:
     ```bash
     python -m http.server 8000
     ```

4. **Login or Create Account**
   - Use your registered email and password
   - Or sign up for a new account if enabled in Supabase

### Using the Application

#### **Editing Content**
1. Click the **Edit** button (blue) in the top control bar
2. The button turns emerald green to indicate edit mode
3. Click any field to edit (left pages only)
4. Changes auto-save to localStorage every keystroke
5. Cloud sync happens automatically (with 1.5s debounce)
6. Click **Edit** again to exit edit mode

#### **Printing**
1. Click the **Print** button to open the print dialog
2. Browser print dialog appears with preview
3. Ensure print settings:
   - Paper: A4 Landscape
   - Margins: Default
   - Background graphics: On
4. Click Print to save as PDF or print to paper

#### **Managing Vocabulary Items**
1. Enter vocabulary words in the 20 vocab fields per page
2. Click the copy button next to any vocab field to copy to clipboard
3. Changes sync to all pages automatically

#### **Logging Out**
1. Click the **Logout** button in the top right
2. If switching accounts, you'll see a warning message
3. Current session ends and draft saves to cloud

## Architecture

### File Structure
```
ContactSystem/
├── index.html                  # Redirect to main app
├── ContactSystem.html          # Main application (HTML + CSS + JS)
├── CLAUDE.md                   # Development guide
└── README.md                   # This file
```

### Data Model

**Editable Fields**
- Pattern: `.editable-field` with `data-key="p{n}-{fieldname}"`
- Example: `p1-title`, `p1-vocab1`, `p5-homework`
- Odd pages (1, 3, 5, 7, 9): Left column - fully editable
- Even pages (2, 4, 6, 8, 10): Right column - read-only mirrors

**Cloud Storage**
- Table: `contact_drafts`
- Columns: `user_id` (UUID), `draft_data` (JSONB)
- Key: User ID (primary key)

### Data Flow

**On Load**
1. Check Supabase auth state
2. Load draft from cloud if authenticated
3. Fallback to localStorage if offline
4. Apply data to all editable fields
5. Sync odd pages to even pages

**On Edit**
1. User changes field content
2. Blur/input event triggers
3. Save to localStorage immediately
4. Debounced Supabase sync (1500ms)
5. Auto-sync odd to even pages

**On Print**
1. Exit edit mode if active
2. Media query hides UI elements
3. Opens browser print dialog
4. Two pages render per A4 sheet

## Development

### Project Structure Details
- All code in single HTML file (no build required)
- HTML: Lines 1-228 (authentication & structure)
- CSS: Lines 9-177 (embedded styles + Tailwind)
- JavaScript: Lines 851+ (logic & API calls)

### Adding New Fields

1. **Add HTML element** with `.editable-field` class
   ```html
   <div class="editable-field" data-key="p1-newfield">Default content</div>
   ```

2. **For new pages**, add to both left and right columns

3. **JavaScript auto-detection**: Field will automatically be:
   - Editable if on odd page and in edit mode
   - Read-only if on even page

### Modifying Styling
- CSS variables: `--primary-blue`, `--light-blue-bg`, `--border-gray`, `--text-main`
- Tailwind CSS classes for responsive design
- Print media query: `@media print` (line 61-67)

### Testing

**Authentication**
```javascript
// Check localStorage for draft
localStorage.getItem('s2_contact_draft')

// Monitor Supabase calls in Network tab
// Check console for error messages
```

**Edit Mode**
- Toggle by clicking Edit button
- Verify contentEditable changes on odd pages
- Check that even pages remain read-only

**Printing**
- Open DevTools → Ctrl+Shift+P → Rendering → Emulate CSS media type: Print
- Verify layout and spacing
- Check dashed cut lines between pages

## Configuration

### Supabase Setup Required

```sql
-- Create contact_drafts table
CREATE TABLE contact_drafts (
  user_id UUID PRIMARY KEY REFERENCES auth.users(id),
  draft_data JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  updated_at TIMESTAMP DEFAULT now()
);

-- Enable Row Level Security
ALTER TABLE contact_drafts ENABLE ROW LEVEL SECURITY;

-- Create policies (if needed)
CREATE POLICY "Users can read own drafts"
  ON contact_drafts FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can update own drafts"
  ON contact_drafts FOR UPDATE
  USING (auth.uid() = user_id);
```

### Environment Variables
- Update Supabase URL in `ContactSystem.html` (line 852)
- Update Supabase anon key in `ContactSystem.html` (line 853)

## Browser Support

- Chrome/Chromium: ✅ Full support
- Firefox: ✅ Full support
- Safari: ✅ Full support (iOS 13+)
- Edge: ✅ Full support
- IE 11: ❌ Not supported

## Performance

- **Load Time**: < 1s (single file)
- **Sync Debounce**: 1500ms to Supabase
- **Field Count**: ~40 editable fields processed efficiently
- **Offline Support**: Full functionality with localStorage fallback

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Can't login** | Verify Supabase credentials; check user exists in auth |
| **Changes not saving** | Check localStorage is enabled; verify network connection |
| **Print layout broken** | Set A4 landscape in print dialog; disable header/footer |
| **Edit button not working** | Refresh page; check browser console for errors |
| **Sync failures** | Check network tab; local draft is preserved as backup |
| **Vocab fields not visible** | Clear browser cache; reload page |

## Recent Updates

- ✨ Concurrent login prevention with user warnings
- 🔧 Logout improvements for session management
- 🐛 Fixed duplicate key errors with UPSERT
- 📋 Copy buttons for all vocabulary fields
- 🎨 Improved UI feedback and status messages

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly (editing, printing, auth)
5. Commit with clear messages
6. Push to your fork
7. Submit a pull request

## License

MIT License - feel free to use and modify for your needs

## Support

For issues or questions:
- Check [CLAUDE.md](CLAUDE.md) for development details
- Review browser console for error messages
- Check Supabase dashboard for data issues

---

**Made with ❤️ for educators by Randolph Chien**

聯絡單管理系統 - 為老師設計的現代化聯絡表單管理工具
