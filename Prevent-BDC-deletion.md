### Prevent Accidental Deletion of BDC

High level workflow for prevention of accidental deletion of BlockDeviceClaims(BDC)

#### NDM Operator (BDC Controller)
1. NDM Operator will be watching all the BDCs
2. If DeletionTimeStamp is set on the BDC, then:

	Controller will check for the no.of finalizers on the BDC, if it has only one finalizer,
	then check whether NDM finalizer is present.

	If yes, the BD will be marked as released and the NDM finalizer will be removed.
3. NDM operator will make sure that, its finalizer is the last one to be removed from the BDC

#### Maya (Owner of BDC)
1. Maya will create the BDC and add a finalizer to it.
2. When pool/localpv is deleted and cleanedup, this finalizer should be removed by maya, and will delete the BDC
3. Once deletion timestamp is set, NDM operator takes up the work.


The above solution will give partial control of how the BDC should be deleted to the owner of BDC. If the user who creates the claim, want to prevent accidental deletion, he can add a finalizer but then it will be the reponsibility of the user to remove it.

This approach also makes sure that BD will be marked as Released only when no one else is using the corresponding BDC.