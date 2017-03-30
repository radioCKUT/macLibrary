# macLibrary
The entire Add to Digital Music Library software suite consists of 2 applications and 1 folder action .workflow.

The applications are set as the default applications for all iTunes compatible sound files, and for PDF files. Their purpose is to prompt users as to whether they with to import files to the Digital Music Library, or open the files normally. Depending on the user’s input the applications open the files with the application of the user’s choosing, or move the files to /Users/library/Music/iTunes/Add to Digital Music Library.


/Users/library/Music/iTunes/Add to Digital Music Library is associated with the Folder Action .workflow file /Users/library/Library/Workflows/Applications/Folder Actions/Add to DLM 3/21/17.workflow 


The following is a summary of the .workflow’s actions; it does not demonstrate the order of operations or the method by which the .workflow operates. It is best to look at the .workflow it in Automator for a clearer understanding of how it functions.

The .workflow file imports all audio files to iTunes, copies the files into iTune’s file architecture on the CKUT Digital Music Library external HDD, moves the original files to the Trash, and tags the new iTunes tracks with the category “Digital Music Library”.

In the case of PDF files the .workflow opens a dialog that prompts the user to wait until the “PDF Adder” script (from dougscripts.com) has launched.  It then launches an iteration of “PDF Adder” found in Macintosh HD/Users/library/Library/iTunes/PDF Adders Workaround/ to open each PDF file added to the folder.  The PDF Adder script prompts the user to enter the relevant information to tag PDFs in iTunes.  The PDF files are then added to iTunes, the files are copied into iTune’s file architecture on the CKUT Digital Music Library external HDD, the original files are moved to the Trash, and the PDFs are tagged in iTunes with the category “Digital Music Library”.


to make change to the .workflow file access Folder Actions Setup by right clicking any folder and selecting “Folder Actions Setup…” at the bottom of the menu. Then select the Add to DML 3:21:17.workflow and click [Edit Workflow] to make changes.  The .workflow file will open in Automator.

The workflow contains two Apple scripts, the text of which is included below for reference.
The rest of the Automator script is composed of various iTunes commands which create playlists, Finder commands that move and delete files.

if installed on another machine all references to file directories in Automator and the AppleScripts themselves will have to be updated to reflect the local file architecture.

Please note that the first AppleScript does not use the 
{
on run {input,parameters}
	—-script
	return {input}
end run 
}
form usually required to run an appleScript from Automator.  This is by design.  Unfortunately I can’t remember why, but I’m pretty sure it doesn’t work when that additional code is inserted.

The two AppleScripts are as follows:

{
(*Set aliases for Add to iTunes Folder, and the folder that contains the PDF Adder apps (unfortunately each app can only handle 1 pdf at a time, so numerous copies are run simultaneously as a work around)*)

set addiTunes to alias "Macintosh HD:Users:library:Music:iTunes:Add to Digital Music Library"
set AddersApps to alias "Macintosh HD:Users:library:Library:iTunes:PDF Adders Workaround"
set theAlertText to "Wait for PDF Adder to launch."
set theAlertMessage to "Please tag the PDF documents you wish to import into iTunes with the appropriate 'Name' 'Artist' and 'Album' tags.

Check the tracks you just imported into iTunes to ensure that everything is spelled correctly.
	
	 When finished click 'Continue' to proceed."

tell application "Finder"
	--set list of PDFs 
	set pdfList to (every file in the entire contents of addiTunes whose name extension is "pdf")
end tell

if pdfList is {} then
	return
else if (count of pdfList) is 1 then
	tell application "Finder"
		open pdfList using application file "Macintosh HD:Users:library:Library:iTunes:Scripts:PDF Adder.app"
	end tell
	display alert theAlertText message theAlertMessage as critical buttons {"Continue"} default button "Continue"
	
else
	set pdfCount to count of pdfList
	tell application "Finder"
		set appList to (items 1 thru pdfCount of AddersApps)
		
		-- overcomes issue with iterating repeat loop trough applist containing 1 item 
		
		--Opens every file in the pdfList with an independent applet stored in the appList
		repeat with a from 1 to count of pdfList
			open item a of pdfList using item a of appList
		end repeat
	end tell
	display alert theAlertText message theAlertMessage as critical buttons {"Continue"} default button "Continue"
end if
}

and 

{
(* This script tags all items in the playlist "New Imports" (All the PDFs and audio files imported to iTunes earlier in the workflow) with the category tag "Digital Music Library" and then deletes the playlist from the iTunes Library.  The imported files will remain as tracks in the main library. *)

on run {input, parameters}
	
	tell application "iTunes"
		activate
		set results to (every track of playlist "New Imports")
		repeat with t in results
			set category of t to ("Digital Music Library")
		end repeat
		delete playlist "New Imports"
	end tell
	
	return input
end run
}
