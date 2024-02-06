Create a new functional component in the frontend/src directory called `NoteSearch.js`
```jsx
import React from 'react';

const NoteSearch = ({ searchQuery, setSearchQuery }) => {
  return (
    <div>
      <input
        type="text"
        placeholder="Search notes by title..."
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        style={{ margin: '20px 0', padding: '10px', width: 'calc(100% - 40px)' }}
      />
      <button onClick={() => setSearchQuery("")} style={{ padding: '10px' }}>Clear Search</button>
    </div>
  );
};

export default NoteSearch;

```

In App.js, add the following:

```jsx
import NoteSearch from "./NoteSearch";
...
const [searchQuery, setSearchQuery] = useState("");
...

const filteredNotes = searchQuery
? notes.filter(note =>
    note.title.toLowerCase().includes(searchQuery.toLowerCase())
  )
: notes;

...
<NoteSearch searchQuery={searchQuery} setSearchQuery={setSearchQuery} />
...

filteredNotes ?
filteredNotes.map((entry) => {
  return (
  <div key={entry._id}>
    <Note
    entry={entry} 
    editNote={editNote} 
    deleteNote={deleteNote}
    />
  </div>
  )
})
```


See the full source code of App.js:

```jsx
import React, {useState, useEffect} from "react"
import './App.css';
import Dialog from "./Dialog";
import Note from "./Note";
import NoteSearch from "./NoteSearch";

function App() {

  // -- Backend-related state --
  const [loading, setLoading] = useState(true)
  const [notes, setNotes] = useState(undefined)

  // -- Dialog props-- 
  const [dialogOpen, setDialogOpen] = useState(false)
  const [dialogNote, setDialogNote] = useState(null)
  const [searchQuery, setSearchQuery] = useState("");
  
  // -- Database interaction functions --
  useEffect(() => {
    const getNotes = async () => {
      try {
        await fetch("http://localhost:4000/getAllNotes")
        .then(async (response) => {
          if (!response.ok) {
            console.log("Served failed:", response.status)
          } else {
              await response.json().then((data) => {
              getNoteState(data.response)
          }) 
          }
        })
      } catch (error) {
        console.log("Fetch function failed:", error)
      } finally {
        setLoading(false)
      }
    }

    getNotes()
  }, [])

  const deleteNote = async (entry) => {
    deleteNoteState(entry._id);

    try {
      const response = await fetch(`http://localhost:4000/deleteNote/${entry._id}`, {
        method: "DELETE",
        headers: {
            "Content-Type": "application/json"
        },
      });
  
      if (!response.ok) {
        console.log("Server failed to delete the note:", response.status);
      }
    } catch (error) {
      console.error("Delete function failed:", error);
    }
  }

  const deleteAllNotes = async () => {
    try {
      await fetch(`http://localhost:4000/deleteAllNotes`, {
        method: "DELETE",
        headers: {
            "Content-Type": "application/json"
        },
      })
      .then(async (response) => {
        if (!response.ok) {
          console.log("Served failed:", response.status)
          alert("Failed to delete all notes!")
        } else {
            await response.json().then(() => {
              deleteAllNotesState()
        }) 
        }
      })
    } catch (error) {
      console.log("Delete function failed:", error)
      alert("Failed to delete all notes!")
    } finally {
      setLoading(false)
    }
  }

  
  // -- Dialog functions --
  const editNote = (entry) => {
    setDialogNote(entry)
    setDialogOpen(true)
  }

  const postNote = () => {
    setDialogNote(null)
    setDialogOpen(true)
  }

  const closeDialog = () => {
    setDialogNote(null)
    setDialogOpen(false)
  }

  // -- State modification functions -- 
  const getNoteState = (data) => {
    setNotes(data)
  }

  const postNoteState = (_id, title, content) => {
    setNotes((prevNotes) => [...prevNotes, {_id, title, content}])
  }

  const deleteNoteState = (_id) => {
    setNotes((prevNotes) => prevNotes.filter((note) => note._id !== _id))
  }

  const deleteAllNotesState = () => {
    setNotes([])
  }

  const patchNoteState = (_id, title, content) => {
    setNotes((prevNotes) => prevNotes.map((note) => {
      if (note._id !== _id) {
        return note
      }

      return (
        {_id, title, content}
      )
    }))
  }


  const filteredNotes = searchQuery
  ? notes.filter(note =>
      note.title.toLowerCase().includes(searchQuery.toLowerCase())
    )
  : notes;

  return (
    <div className="App">
      <header className="App-header">
        <div style={dialogOpen ? AppStyle.dimBackground : {}}>
          <h1 style={AppStyle.title}>QuirkNotes</h1>
          <h4 style={AppStyle.text}>The best note-taking app ever </h4>

          <NoteSearch searchQuery={searchQuery} setSearchQuery={setSearchQuery} />

          <div style={AppStyle.notesSection}>
            {loading ?
            <>Loading...</>
            : 
            filteredNotes ?
            filteredNotes.map((entry) => {
              return (
              <div key={entry._id}>
                <Note
                entry={entry} 
                editNote={editNote} 
                deleteNote={deleteNote}
                />
              </div>
              )
            })
            :
            <div style={AppStyle.notesError}>
              Something has gone horribly wrong!
              We can't get the notes!
            </div>
            }
          </div>

          <button onClick={postNote}>Post Note</button>
          {notes && notes.length > 0 && 
          <button
              onClick={deleteAllNotes}
              >
              Delete All Notes
          </button>}

        </div>

        <Dialog
          open={dialogOpen}
          initialNote={dialogNote}
          closeDialog={closeDialog}
          postNote={postNoteState}
          patchNote={patchNoteState}
          />

      </header>
    </div>
  );
}

export default App;

const AppStyle = {
  dimBackground: {
    opacity: "20%", 
    pointerEvents: "none"
  },
  notesSection: {
    display: 'flex',
    flexWrap: 'wrap',
    justifyContent: "center"
  },
  notesError: {color: "red"},
  title: {
    margin: "0px"
  }, 
  text: {
    margin: "0px"
  }
}


```
