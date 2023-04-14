# 第五课作业

1. **列出3个常用的宏、3个常用的存储数据结构。**
   
* 三种常用的宏：frame_support::pallet、pallet::hooks、construct_runtime
* 三种常用的存储数据结构：单值存储（StorageValue）、映射（StorageMap）、双键映射（StorageDoubleMap）
   
2. **实现存证模块的功能，包括：创建存证；撤销存证。**
   
   ```rust
   // Dispatchable functions allow users to interact with the pallet and invoke state changes.
   // These functions materialize as "extrinsics", which are often compared to transactions.
   // Dispatchable functions must be annotated with a weight and must return a DispatchResult.
   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(0)]
   	#[pallet::call_index(1)]
   	pub fn create_claim(origin: OriginFor<T>, claim: T::Hash) -> DispatchResult {
   		// Check that the extrinsic was signed and get the signer.
   		// This function will return an error if the extrinsic is not signed.
   		let sender = ensure_signed(origin)?;
   
   		// Verify that the specified claim has not already been stored.
   		ensure!(!Claims::<T>::contains_key(&claim), Error::<T>::AlreadyClaimed);
   
   		// Get the block number from the FRAME System pallet.
   		let current_block = <frame_system::Pallet<T>>::block_number();
   
   		// Store the claim with the sender and block number.
   		Claims::<T>::insert(&claim, (&sender, current_block));
   
   		// Emit an event that the claim was created.
   		Self::deposit_event(Event::ClaimCreated { who: sender, claim });
           
   		Ok(())
   	}
   
   	#[pallet::weight(0)]
   	#[pallet::call_index(2)]
   	pub fn revoke_claim(origin: OriginFor<T>, claim: T::Hash) -> DispatchResult {
   		// Check that the extrinsic was signed and get the signer.
   		// This function will return an error if the extrinsic is not signed.
   		let sender = ensure_signed(origin)?;
   
   		// Get owner of the claim, if none return an error.
   		let (owner, _) = Claims::<T>::get(&claim).ok_or(Error::<T>::NoSuchClaim)?;
   
   		// Verify that sender of the current call is the claim owner.
   		ensure!(sender == owner, Error::<T>::NotClaimOwner);
   
   		// Remove claim from storage.
   		Claims::<T>::remove(&claim);
   
   		// Emit an event that the claim was erased.
   		Self::deposit_event(Event::ClaimRevoked { who: sender, claim });
   		Ok(())
   	}
   ```


3. **为存证模块添加新的功能，转移存证，接收两个参数，一个是包含的哈希值，另一个是存证的接收账户地址。**
   
   ```rust
   #[pallet::weight(0)]
   #[pallet::call_index(3)]
   pub fn transfer_claim(
   	origin: OriginFor<T>, 
       claim: T::Hash, 
       receiver: T::AccountId
   ) -> DispatchResult {
       // Check that the extrinsic was signed and get the signer.
   	// This function will return an error if the extrinsic is not signed.
       let sender = ensure_signed(origin)?;
       
       // Get owner of the claim, if none return an error.
       let (owner, _) = Claims::<T>::get(&claim).ok_or(Error::<T>::NoSuchClaim)?;
   
    // Verify that sender of the current call is the claim owner.
       ensure!(sender == owner, Error::<T>::NotClaimOwner);
   
       // Get the block number from the FRAME System pallet.
       let current_block = <frame_system::Pallet<T>>::block_number();
   
       // Remove claim from storage.
       Claims::<T>::remove(&claim);
   
       // Store the claim with the sender and block number.
       Claims::<T>::insert(&claim, (&receiver, current_block));
   
       // Emit an event that the claim was erased.
       Self::deposit_event(Event::ClaimTransfered { who: sender, claim, receiver });
   
       Ok(())
   }
   ```
   
   创建存证：
   
   ![createClaim](./imgs/createClaim.png)
   
   存证转移：
   
   ![transferClaim](./imgs/transferClaim.png)
   
   存证销毁：
   
   ![revokeClaim](./imgs/revokeClaim.png)



