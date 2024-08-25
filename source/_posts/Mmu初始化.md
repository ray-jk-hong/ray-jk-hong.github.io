
---
title: Mmu初始化
categories: 
- Linux MM
tags:
- Linux MM
---

## __cpu_setup
__cpu_setup函数[arch/arm64/mm/proc.S]

```c
   393	/*
   394	 *	__cpu_setup
   395	 *
   396	 *	Initialise the processor for turning the MMU on.
   397	 *
   398	 * Input:
   399	 *	x0 - actual number of VA bits (ignored unless VA_BITS > 48)
   400	 * Output:
   401	 *	Return in x0 the value of the SCTLR_EL1 register.
   402	 */
   403		.pushsection ".idmap.text", "a"
   404	SYM_FUNC_START(__cpu_setup)
   405		tlbi	vmalle1				// Invalidate local TLB
   406		dsb	nsh
   407	
   408		mov	x1, #3 << 20
   409		msr	cpacr_el1, x1			// Enable FP/ASIMD
   410		mov	x1, #1 << 12			// Reset mdscr_el1 and disable
   411		msr	mdscr_el1, x1			// access to the DCC from EL0
   412		isb					// Unmask debug exceptions now,
   413		enable_dbg				// since this is per-cpu
   414		reset_pmuserenr_el0 x1			// Disable PMU access from EL0
   415		reset_amuserenr_el0 x1			// Disable AMU access from EL0
   416	
   417		/*
   418		 * Default values for VMSA control registers. These will be adjusted
   419		 * below depending on detected CPU features.
   420		 */
   421		mair	.req	x17
   422		tcr	.req	x16
   423		mov_q	mair, MAIR_EL1_SET
   424		mov_q	tcr, TCR_TxSZ(VA_BITS) | TCR_CACHE_FLAGS | TCR_SMP_FLAGS | \
   425				TCR_TG_FLAGS | TCR_KASLR_FLAGS | TCR_ASID16 | \
   426				TCR_TBI0 | TCR_A1 | TCR_KASAN_SW_FLAGS | TCR_MTE_FLAGS
   427	
   428		tcr_clear_errata_bits tcr, x9, x5
   429	
   430	#ifdef CONFIG_ARM64_VA_BITS_52
   431		sub		x9, xzr, x0
   432		add		x9, x9, #64
   433		tcr_set_t1sz	tcr, x9
   434	#else
   435		idmap_get_t0sz	x9
   436	#endif
   437		tcr_set_t0sz	tcr, x9
   438	
   439		/*
   440		 * Set the IPS bits in TCR_EL1.
   441		 */
   442		tcr_compute_pa_size tcr, #TCR_IPS_SHIFT, x5, x6
   443	#ifdef CONFIG_ARM64_HW_AFDBM
   444		/*
   445		 * Enable hardware update of the Access Flags bit.
   446		 * Hardware dirty bit management is enabled later,
   447		 * via capabilities.
   448		 */
   449		mrs	x9, ID_AA64MMFR1_EL1
   450		and	x9, x9, ID_AA64MMFR1_EL1_HAFDBS_MASK
   451		cbz	x9, 1f
   452		orr	tcr, tcr, #TCR_HA		// hardware Access flag update
   453	1:
   454	#endif	/* CONFIG_ARM64_HW_AFDBM */
   455		msr	mair_el1, mair
   456		msr	tcr_el1, tcr
   457	
   458		mrs_s	x1, SYS_ID_AA64MMFR3_EL1
   459		ubfx	x1, x1, #ID_AA64MMFR3_EL1_S1PIE_SHIFT, #4
   460		cbz	x1, .Lskip_indirection
   461	
   462		mov_q	x0, PIE_E0
   463		msr	REG_PIRE0_EL1, x0
   464		mov_q	x0, PIE_E1
   465		msr	REG_PIR_EL1, x0
   466	
   467		mov	x0, TCR2_EL1x_PIE
   468		msr	REG_TCR2_EL1, x0
   469	
   470	.Lskip_indirection:
   471	
   472		/*
   473		 * Prepare SCTLR
   474		 */
   475		mov_q	x0, INIT_SCTLR_EL1_MMU_ON
   476		ret					// return to head.S
   477	
   478		.unreq	mair
   479		.unreq	tcr
   480	SYM_FUNC_END(__cpu_setup)
```

## __enable_mmu
```c 
/* 
  1  * Enable the MMU.
  2  * 
  3  *  x0  = SCTLR_EL1 value for turning on the MMU. 
  4  *  x1  = TTBR1_EL1 value 
  5  *  x2  = ID map root table address 
  6  * 
  7  * Returns to the caller via x30/lr. This requires the caller to be covered 
  8  * by the .idmap.text section. 
  9  *
 10  * Checks if the selected granule size is supported by the CPU. 
 11  * If it isn't, park the CPU 
 12  */ 
 13     .section ".idmap.text","a" 
 14 SYM_FUNC_START(__enable_mmu) 
 15     mrs x3, ID_AA64MMFR0_EL1 
 16     ubfx    x3, x3, #ID_AA64MMFR0_EL1_TGRAN_SHIFT, 4 
 17     cmp     x3, #ID_AA64MMFR0_EL1_TGRAN_SUPPORTED_MIN 
 18     b.lt    __no_granule_support 
 19     cmp     x3, #ID_AA64MMFR0_EL1_TGRAN_SUPPORTED_MAX 
 20     b.gt    __no_granule_support 
 21     phys_to_ttbr x2, x2 
 22     msr ttbr0_el1, x2           // load TTBR0 
 23     load_ttbr1 x1, x1, x3 
 24  
 25     set_sctlr_el1   x0 
 26  
 27     ret 
 28 SYM_FUNC_END(__enable_mmu) 
```
