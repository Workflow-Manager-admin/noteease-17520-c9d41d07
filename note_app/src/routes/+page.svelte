<script>
  // PUBLIC_INTERFACE
  // Main Container for NoteEase: manages note list, modal/editor, search, formatting (bold/italic/underline), and CRUD logic.

  // No imports needed


  // Note model structure
  let notes = [
    // Initial sample notes for demonstration
    {
      id: 1,
      title: "Welcome to NoteEase",
      content: "<b>This is your first note!</b> Use the editor to format your notes.",
      createdAt: new Date().toISOString(),
      editedAt: new Date().toISOString(),
    },
    {
      id: 2,
      title: "Formatting Tips",
      content: "You can use <b>Bold</b>, <i>Italic</i>, and <u>Underline</u>.",
      createdAt: new Date().toISOString(),
      editedAt: new Date().toISOString(),
    }
  ];

  // For note modal (creation or editing)
  let showModal = false;
  let isEditing = false;
  let editNoteId = null;
  let modalTitle = "";
  let modalContent = "";

  // Search term state
  let searchTerm = "";

  // PUBLIC_INTERFACE
  function openModalToCreate() {
    isEditing = false;
    editNoteId = null;
    modalTitle = "";
    modalContent = "";
    showModal = true;
  }

  // PUBLIC_INTERFACE
  function openModalToEdit(note) {
    isEditing = true;
    editNoteId = note.id;
    modalTitle = note.title;
    modalContent = note.content;
    showModal = true;
  }

  // PUBLIC_INTERFACE
  function saveNote() {
    if (!modalTitle.trim() && !modalContent.trim()) {
      // Prevent saving empty notes
      return;
    }

    if (isEditing && editNoteId !== null) {
      notes = notes.map((n) =>
        n.id === editNoteId
          ? {
              ...n,
              title: modalTitle,
              content: modalContent,
              editedAt: new Date().toISOString()
            }
          : n
      );
    } else {
      const newNote = {
        id: Date.now(), // crude unique id
        title: modalTitle,
        content: modalContent,
        createdAt: new Date().toISOString(),
        editedAt: new Date().toISOString()
      };
      notes = [newNote, ...notes];
    }
    closeModal();
  }

  // PUBLIC_INTERFACE
  function deleteNote(noteId) {
    if (confirm("Delete this note?")) {
      notes = notes.filter((n) => n.id !== noteId);
      if (isEditing && editNoteId === noteId) {
        closeModal();
      }
    }
  }

  // PUBLIC_INTERFACE
  function closeModal() {
    showModal = false;
    isEditing = false;
    modalTitle = "";
    modalContent = "";
    editNoteId = null;
  }

  // PUBLIC_INTERFACE
  function insertFormatting(tag) {
    // Inserts formatting tags into the contenteditable modalContent
    let textarea = document.getElementById("note-content-editable");
    if (!textarea) return;

    // For browser compatibility, use execCommand (deprecated but usable for this context)
    textarea.focus();
    document.execCommand(tag, false);
    // Sync innerHTML
    modalContent = textarea.innerHTML;
  }

  // Handler for getting snippet from HTML content
  // PUBLIC_INTERFACE
  function getSnippet(html, maxLength = 60) {
    // Strip HTML to plain text for snippet
    let tmp = document.createElement("div");
    tmp.innerHTML = html;
    let txt = tmp.innerText;
    return txt.length > maxLength ? txt.substring(0, maxLength) + "…" : txt;
  }

  // Filter notes based on searchTerm (case-insensitive, in title or content)
  $: filteredNotes = searchTerm.trim().length
    ? notes.filter(
        (n) => {
          const div = document.createElement("div");
          div.innerHTML = n.content;
          const plain = div.textContent || div.innerText || "";
          return (
            n.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
            plain.toLowerCase().includes(searchTerm.toLowerCase())
          );
        }
      )
    : notes;

  // Sync contenteditable innerHTML to modalContent
  function handleContentInput(e) {
    modalContent = e.target.innerHTML;
  }
</script>

<style>
  :global(body) {
    font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
    background: #F9F9F9;
    color: #222222;
  }
  .container {
    background: #FFFFFF;
    max-width: 420px;
    margin: 3rem auto;
    border-radius: 14px;
    box-shadow: 0 3px 24px 0 rgba(50, 120, 180, 0.09);
    padding: 0 0 2.5rem 0;
    min-height: 75vh;
    border: 1px solid #E6E6E6;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }
  .main-header {
    font-weight: 700;
    font-size: 2rem;
    color: #4A90E2;
    padding: 2rem 2rem 0.2rem 2rem;
    text-align: left;
    letter-spacing: 0.01em;
  }
  .search-bar {
    padding: 0 2rem 0.8rem 2rem;
  }
  .search-bar input {
    width: 100%;
    padding: 0.7rem 1.2rem;
    font-size: 1rem;
    border: 1px solid #E6E6E6;
    border-radius: 48px;
    outline: none;
    background: #F9F9F9;
    color: #222222;
    transition: box-shadow 0.2s;
  }
  .search-bar input:focus {
    box-shadow: 0 0 0 2px #4A90E233;
  }
  .note-list {
    flex: 1;
    overflow-y: auto;
    margin: 0 2rem;
  }
  .note-item {
    display: flex;
    flex-direction: column;
    padding: 1.1rem 1.2rem 1.1rem 1rem;
    border-bottom: 1px solid #E6E6E6;
    margin-bottom: 0.3rem;
    background: #FFFFFF;
    border-radius: 9px;
    transition: box-shadow 0.11s;
    position: relative;
    cursor: pointer;
  }
  .note-item:hover {
    background: #F9F9F9;
    box-shadow: 0 2px 9px 2px #4a90e203;
  }
  .note-title {
    font-weight: 600;
    font-size: 1.07rem;
    color: #4A90E2;
    margin-bottom: 0.09rem;
    word-break: break-all;
  }
  .note-snippet {
    color: #333;
    font-size: 0.99rem;
    opacity: 0.80;
    margin-bottom: 0.3rem;
    word-break: break-word;
  }
  .note-actions {
    position: absolute;
    right: 1.2rem;
    top: 0.92rem;
    display: flex;
    gap: 0.5rem;
  }
  .note-actions button {
    background: none;
    border: none;
    color: #F5A623;
    font-size: 1.17rem;
    cursor: pointer;
  }
  .fab {
    position: fixed;
    right: calc(10vw + 1rem);
    bottom: 3.8rem;
    z-index: 99;
    background: #4A90E2;
    color: #FFFFFF;
    width: 60px;
    height: 60px;
    border-radius: 50%;
    font-size: 2.3rem;
    box-shadow: 0 4px 18px 0 rgba(74, 144, 226, 0.11);
    border: none;
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: background 0.2s;
  }
  .fab:hover {
    background: #F5A623;
    color: #4A90E2;
  }
  /* Modal styles */
  .modal-overlay {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: #2227;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 200;
  }
  .modal {
    background: #FFFFFF;
    border-radius: 1rem;
    box-shadow: 0 11px 40px 0 #4A90E222;
    max-width: 97vw;
    width: 384px;
    min-height: 330px;
    padding: 2rem 1.6rem 1.3rem 1.6rem;
    position: relative;
    display: flex;
    flex-direction: column;
  }
  .modal-header {
    font-size: 1.14rem;
    font-weight: 600;
    color: #4A90E2;
    margin-bottom: 0.7rem;
    text-align: left;
  }
  .modal-close {
    position: absolute;
    right: 1rem;
    top: 0.7rem;
    font-size: 1.5rem;
    color: #BBB;
    background: none;
    border: none;
    cursor: pointer;
    z-index: 11;
  }
  .modal-form input {
    width: 100%;
    font-size: 1rem;
    padding: 0.55rem 0.95rem;
    border: 1px solid #E6E6E6;
    border-radius: 7px;
    background: #F9F9F9;
    color: #222222;
    margin-bottom: 0.65rem;
    outline: none;
    box-sizing: border-box;
  }
  .modal-form input:focus {
    border: 1.7px solid #4A90E2;
  }
  .modal-form .format-toolbar {
    display: flex;
    gap: 0.6rem;
    margin-bottom: 0.45rem;
  }
  .modal-form .format-toolbar button {
    background: none;
    border: none;
    color: #4A90E2;
    font-size: 1.11rem;
    font-weight: 600;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.12s;
    padding: 0.18rem 0.21rem;
  }
  .modal-form .format-toolbar button:hover {
    background: #9BB9D7;
    color: #F5A623;
  }
  .modal-form .content-editable {
    border: 1.1px solid #E6E6E6;
    border-radius: 8px;
    min-height: 95px;
    padding: 0.7rem 0.9rem;
    font-size: 1rem;
    background: #F9F9F9;
    color: #222222;
    outline: none;
    margin-bottom: 0.6rem;
    transition: border 0.13s;
  }
  .modal-form .content-editable:focus {
    border: 1.7px solid #4A90E2;
  }
  .modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 0.7rem;
    margin-top: 1.1rem;
  }
  .modal-footer button {
    border: none;
    border-radius: 7px;
    font-size: 1rem;
    padding: 0.54rem 1.18rem;
    cursor: pointer;
    font-weight: 500;
    background: #4A90E2;
    color: #FFFFFF;
    transition: background 0.17s;
  }
  .modal-footer button.delete {
    background: #FFBBBB;
    color: #A00;
  }
  .modal-footer button.cancel {
    background: #F9F9F9;
    color: #4A90E2;
    border: 1px solid #E6E6E6;
  }
  .empty-state {
    text-align: center;
    padding: 4rem 1.7rem 1.7rem 1.7rem;
    color: #AAA;
    font-size: 1.17rem;
    font-style: italic;
  }
</style>

<div class="container">
  <div class="main-header">
    NoteEase
  </div>
  <div class="search-bar">
    <input
      type="text"
      placeholder="Search notes…"
      bind:value={searchTerm}
      aria-label="Search notes"
      autocomplete="off"
    />
  </div>
  <div class="note-list" aria-live="polite">
    {#if filteredNotes.length > 0}
      {#each filteredNotes as note (note.id)}
        <div class="note-item" on:click={() => openModalToEdit(note)} tabindex="0" role="listitem" aria-label={"Note: " + note.title}>
          <div class="note-title">{note.title}</div>
          <div class="note-snippet">{getSnippet(note.content, 58)}</div>
          <div class="note-actions" on:click|stopPropagation>
            <button aria-label="Edit note" title="Edit" on:click={() => openModalToEdit(note)}>
              ✏️
            </button>
            <button aria-label="Delete note" title="Delete" on:click={() => deleteNote(note.id)}>
              🗑️
            </button>
          </div>
        </div>
      {/each}
    {:else}
      <div class="empty-state">
        {searchTerm ? "No matching notes found." : "No notes. Tap + to create your first note!"}
      </div>
    {/if}
  </div>

  <!-- Floating Action Button -->
  <button class="fab" aria-label="Add Note" on:click={openModalToCreate} title="Add Note">
    +
  </button>

  {#if showModal}
    <div class="modal-overlay" on:click={closeModal}>
      <div class="modal" on:click|stopPropagation>
        <button class="modal-close" aria-label="Close modal" on:click={closeModal}>&times;</button>
        <div class="modal-header">
          {isEditing ? "Edit Note" : "New Note"}
        </div>
        <form class="modal-form" on:submit|preventDefault={saveNote}>
          <input
            type="text"
            placeholder="Title"
            bind:value={modalTitle}
            maxlength="100"
            aria-label="Note title"
            required
          />
          <!-- Formatting Toolbar -->
          <div class="format-toolbar" aria-label="Formatting toolbar">
            <button type="button" title="Bold" aria-label="Bold" on:click={() => insertFormatting('bold')}><b>B</b></button>
            <button type="button" title="Italic" aria-label="Italic" on:click={() => insertFormatting('italic')}><i>I</i></button>
            <button type="button" title="Underline" aria-label="Underline" on:click={() => insertFormatting('underline')}><u>U</u></button>
          </div>
          <!-- Content Editable -->
          <div
            id="note-content-editable"
            class="content-editable"
            contenteditable="true"
            bind:innerHTML={modalContent}
            on:input={handleContentInput}
            spellcheck="true"
            aria-label="Note content"
            role="textbox"
            style="min-height:95px;"
          ></div>
          <div class="modal-footer">
            {#if isEditing}
              <button type="button" class="delete" on:click={() => deleteNote(editNoteId)}>Delete</button>
            {/if}
            <button type="button" class="cancel" on:click={closeModal}>Cancel</button>
            <button type="submit">{isEditing ? "Save" : "Add"}</button>
          </div>
        </form>
      </div>
    </div>
  {/if}
</div>
