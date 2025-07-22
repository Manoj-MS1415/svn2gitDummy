		                                                         SVN to Git Migration
⦁	Step 1 : Create svn repo and get ready with svn protocol based serve

		e.g.: svnserve -d -c C:/Users/MS1415/Document/svn_archive 

		     This creates svn://localhost through which can access the local svn repo. This creates a demon process.


⦁	Step 2 : Extract users in svn and store it in some file(svn_authors.txt)

		Run this in powershell

			 svn log --quiet svn://localhost |
			 Select-String '^r' |
			 ForEach-Object { ($_ -split '\s+')[2] } |
			 Sort-Object -Unique |
			 Set-Content -Path "svn-authors.txt"

		In that file manually update the user names as follow
		e.g.: MS1415 = Manoj S <manoj.s@commscope.com>


⦁	Step 3 : Converting svn to git (Run in git bash from here)

	If svn folder is standard

		git svn clone svn://localhost \
		--trunk=trunk\
  		--branches=branches\
		--tags=tags \
		--no-metadata \
		--authors-file=svn_authors.txt \
		repo-git
	
	If svn folder is standard
		
		git svn clone svn://localhost \
		--no-metadata \
		--stdlayout \
		--authors-file=svn-authors.txt \
		repo-git
		
	Wait until migration completes (it my take some time).


⦁	Step 4 : Converting svn tags to git tags (git assumes it as branch)

		Move to git repo
			cd repo_name

		First get all remotes using the command
			git branch -r
	
		Use the following command to change
		for tagref in $(git branch -r | grep 'origin/tags/' | sed 's|origin/||'); do
		tagname=$(echo "$tagref" | sed 's|tags/||')
		git checkout -q "origin/$tagref"
		git tag -a "$tagname" -m "Converted tag $tagname from SVN"
	      done

		Delete remote tracking tags.
		 for branch in $(git branch -r | grep 'origin/tags/' | sed 's|origin/||'); do
		git branch -rd "origin/$branch"
	      done

		Use the command 
			git tag 
		to view tags
		
		We can checkout this tags to new branch for any further updates by using command 
		git checkout -b branch_name tagName


⦁	Step 5 : Converting remote-tracking branches to git branches 

		For converting to branches
		for remote in $(git branch -r | grep -v 'origin/HEAD' | grep -v 'tags/'); do
		local_branch=${remote#origin/}
		git checkout -b "$local_branch" "$remote"
		git branch -rd "$remote"
	      done

		Option : If want to remove extra branch (master which was created in the beginning)
			git branch -d master
		
		Option : If we want to rename branch name trunk to main 
			git branch -m trunk main

		If we don't need remote reference, we can clean up completely using command
			git remote remove origin


		Lastly we can check all branches using command 
			git branch
		and we can switch by using command
			git switch branch_name.


⦁	Step 6 : Optionally pushing to remote i.e. GitHub

		 Add remote
			git remote add origin https://github.com/Manoj-MS1415/svn2gitDummy.git

		 Push all and tags
			git push --all origin
			git push --tags origin

