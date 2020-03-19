# OSH-Medical-PROF
OSH Medical Professional Claims extract

/*   
*      filename: get_osh_medical_professional_claims.sql
*                
*    created by: T. Blakemore
*    created on: 20191204
*   modified by: T. Blakemore
*   modified on: 20200224
*
*   description: This query pulls the MA medical professional claims for Oak Street.
*
*/


/*	STEP 0:	Drop Temp Tables
*
*	If any temp tables built by this query are already in existence, drop them
*	here to avoid getting an "Object already exists" error.  This step is 
*	unnecessary when running through an automated interface, but facilitates 
*	running the query manually if ever needed. 
*
*/

IF OBJECT_ID('tempdb..#members') IS NOT NULL
    DROP TABLE #members;

IF OBJECT_ID('tempdb..#medical_claims') IS NOT NULL
    DROP TABLE #medical_claims;

IF OBJECT_ID('tempdb..#for_output') IS NOT NULL
    DROP TABLE #for_output;



/*	STEP 1:	Identify Qualifying Members
*
*	In this step we are going to begin by identifying any members that would should
*	be included in any reporting to Oak Street.
*/

SELECT	DISTINCT e.BRIGHT_MEMBER_ID
		,e.MBI
  INTO	#members
  FROM	dbo.SD_ELIGIBILITY AS e
 WHERE	e.BENEFIT_TYPE = 'MEDICAL'	 	
--TODO: Need to remove 'AZ' from this list before using in Prod.  This is here simply for testing purposes.
   AND	e.MARKET IN ('OH-UHH', 'TN-BMH', 'IL-AMTA', 'AZ')	
   AND	e.LOB ='MA';

-- VALIDATION: Uncomment the following line to view the contents of the temp table.
--SELECT COUNT(*) FROM #members;




/* STEP 2: Pull Associated Medical Claims
*
*	We are now going to pull in any medical claims associated with the qualified members.
*
*	Note:	BRIGHT_MEMBER_ID is supposed to remain consisten even if someone moves to a different
*			Market.  Because of this we WILL need to reapply the Market filter at the Claim level,
*			but we gain effciency by only having to apply that filter to a much smaller set of
*			data (because we are initially only pulling claims for a subset of members).
*	TODO:
*			Any field listed below prefixed with an "x" can be ignored.
*			Any field listed below that is currently commented out and prefixed with something 
*			other than an "x" needs to be mapped/joined appropriately here and in output. 
*
*/

SELECT	cm.BRIGHT_MEMBER_ID
	  ,cm.PATIENT_FIRST_NAME
	  ,cm.PATIENT_MIDDLE_NAME
	  ,cm.PATIENT_LAST_NAME
	  ,m.MBI
	  --,x.MedicaidNumber  -- Currently not supporting this; consider this a placeholder for future enhancement.
	  ,cm.PLAN_ID 
	  --,e.PBP  
	  --,cm.PriorAuthorizationNumber
      ,cm.BRIGHT_CLAIM_ID
	  ,cm.CLAIM_LINE
	  --,cm.QNXTProviderID
	  ,cm.RENDER_PROV_NPI
	  ,cm.RENDER_PROV_FIRST_NAME
	  ,cm.RENDER_PROV_LAST_NAME
	  --cm.REND_PROV_SPEC
	  ,cm.SERVICE_START_DT
      ,cm.SERVICE_END_DT
      ,cm.PAID_DT
	  ,cm.PROCEDURE_CODE
	  ,cm.MODIFIER_CODE_1
      ,cm.MODIFIER_CODE_2
      ,cm.MODIFIER_CODE_3
      ,cm.MODIFIER_CODE_4
      --,x.MOD5  -- Currently not supporting this; consider this a placeholder for future enhancement.
      --,x.MOD6  -- Currently not supporting this; consider this a placeholder for future enhancement.
      ,cm.PLACE_OF_SERVICE_CODE
	  ,cm.CLAIM_STATUS 
	  ,cm.REMARK_DESCR_1
	  ,cm.REMARK_DESCR_2
	  ,cm.REMARK_DESCR_3
	  ,cm.REMARK_DESCR_4
	  --,cm.HasOtherHealthBenefitPlan
	  --,x.NETWORK  -- Currently not supporting this; consider this a placeholder for future enhancement.
      ,cm.PPO_TYPE
	  --,cm.QNXTProviderID
	  ,cm.BILLING_PROV_NPI
	  ,cm.BILLING_PROV_GROUP_NAME
	  ,cm.BILLING_PROV_LAST_NAME
	  ,cm.BILLING_PROV_FIRST_NAME
	  --,cm.ProviderSpecialty
	  ,cm.ICD_TYPE
	  ,cm.DIAGNOSIS_CODE_1
      ,cm.DIAGNOSIS_CODE_2
      ,cm.DIAGNOSIS_CODE_3
      ,cm.DIAGNOSIS_CODE_4
      ,cm.DIAGNOSIS_CODE_5
      ,cm.DIAGNOSIS_CODE_6
      ,cm.DIAGNOSIS_CODE_7
      ,cm.DIAGNOSIS_CODE_8
      ,cm.DIAGNOSIS_CODE_9
      ,cm.DIAGNOSIS_CODE_10
      ,cm.DIAGNOSIS_CODE_11
      ,cm.DIAGNOSIS_CODE_12
      ,cm.DIAGNOSIS_CODE_13
      ,cm.DIAGNOSIS_CODE_14
      ,cm.DIAGNOSIS_CODE_15
      ,cm.DIAGNOSIS_CODE_16
      ,cm.DIAGNOSIS_CODE_17
      ,cm.DIAGNOSIS_CODE_18
      ,cm.DIAGNOSIS_CODE_19
      ,cm.DIAGNOSIS_CODE_20
      ,cm.DIAGNOSIS_CODE_21
      ,cm.DIAGNOSIS_CODE_22
      ,cm.DIAGNOSIS_CODE_23
      ,cm.DIAGNOSIS_CODE_24
      ,cm.DIAGNOSIS_CODE_25
	  --,x.TOS  -- Currently not supporting this; consider this a placeholder for future enhancement.
	  ,cm.BILLED_AMT
	  ,cm.PROCESSED_AMT
	  ,cm.ALLOWED_AMT
	  ,cm.PAID_AMT
      ,cm.COINSURANCE_AMT
	  ,cm.COPAY_AMT
      ,cm.DEDUCTIBLE_AMT
      ,cm.UNITS_BILLED
      ,cm.UNITS_PROCESSED
	  --,x.HOSPICE_FLAG  -- Currently not supporting this; consider this a placeholder for future enhancement.
	  ,cm.REVERSAL_CLAIM_KEY
	  ,cm.ORIGINAL_BRIGHT_CLAIM_ID
	  --,x.MEDICARE_INDICATOR  -- Currently not supporting this; consider this a placeholder for future enhancement.


	  --,cm.CLAIM_TYPE	--VALIDATION that ONLY Professional claims are being pulled.	


  INTO	#medical_claims
  FROM	#members AS m
		INNER JOIN SD_CLAIM_MEDICAL AS cm
			ON	(m.BRIGHT_MEMBER_ID = cm.BRIGHT_MEMBER_ID)
--TODO: Need to remove 'AZ' from this list before using in Prod.  This is here simply for testing purposes.
 WHERE	cm.MARKET IN ('OH-UHH', 'TN-BMH', 'IL-AMTA', 'AZ') 
		and cm.CLAIM_TYPE = 'Professional'

-- VALIDATION: Uncomment the following line to view the contents of the temp table.
--SELECT * FROM #medical_claims;




/* STEP 3: Build Resultset for Output
*
*	In this step we are going to build a resultset ready for output.  This will be a temp table
*	with fields that are named in terms that Oak Street wants, in the correct order, and with appropriate
*	formatting on the datas.
*/
SELECT cm.BRIGHT_MEMBER_ID AS "HP_SUBSCRIBER_ID"
	  ,COALESCE(cm.PATIENT_FIRST_NAME, '') AS "FIRST_NM"
	  ,COALESCE(cm.PATIENT_MIDDLE_NAME, '') AS "MIDDLE_NM"
	  ,COALESCE(cm.PATIENT_LAST_NAME, '') AS "LAST_NM"
	  ,COALESCE(m.MBI, '') AS "MEDICARE_ID"
	  ,'' AS "MEDICAID_ID"  -- Currently not supporting this; consider this a placeholder for future enhancement.
	  ,COALESCE(cm.PLAN_ID, '') AS "PLAN_ID"
	  --,COALESCE(SUBSTRING(e.PLAN_ID, 6, 3), '') AS "PBP"
	  --,COALESCE(cm.PriorAuthorizationNumber, '') AS "AUTH_ID"
	  ,COALESCE(cm.BRIGHT_CLAIM_ID, '') AS "CLAIM_ID"
	  ,COALESCE(cm.CLAIM_LINE, '') AS "LINE_NO"
	  --,COALESCE(cm.QNXTProviderID, '') AS "REND_PROV_ID"
	  ,COALESCE(cm.RENDER_PROV_NPI, '') AS "REND_NPI"

--TODO: RENDERING below-Eliminate comma when no first name follows
	  ,CONCAT(cm.RENDER_PROV_LAST_NAME,', ', cm.RENDER_PROV_FIRST_NAME) AS "REND_NM"
--Is the syntax below efficient?
	  ,SUBSTRING(
			ISNULL(','+nullif(ltrim(cm.RENDER_PROV_LAST_NAME),''), '')+
			ISNULL(','+nullif(ltrim(cm.RENDER_PROV_FIRST_NAME),''), ''), 2, 1000) AS "REND_NM2"  --TODO-remove # from NM

	  ,'' AS "REND_PROV_SPEC"
	  ,COALESCE(CONVERT(VARCHAR(8), cm.SERVICE_START_DT, 112), '') AS "FROM_DT"
	  ,COALESCE(CONVERT(VARCHAR(8), cm.SERVICE_END_DT, 112), '') AS "THRU_DT"
	  ,COALESCE(CONVERT(VARCHAR(8), cm.PAID_DT, 112), '') AS "PAID_DT"
	  ,COALESCE(cm.PROCEDURE_CODE, '') AS "PROC_CD"
	  ,COALESCE(cm.MODIFIER_CODE_1, '') AS "MOD1"
      ,COALESCE(cm.MODIFIER_CODE_2, '') AS "MOD2"
      ,COALESCE(cm.MODIFIER_CODE_3, '') AS "MOD3"
      ,COALESCE(cm.MODIFIER_CODE_4, '') AS "MOD4"
      ,'' AS "MOD5" -- Currently not supporting this; consider this a placeholder for future enhancement.
      ,'' AS "MOD6" -- Currently not supporting this; consider this a placeholder for future enhancement.
      ,cm.PLACE_OF_SERVICE_CODE AS "POS"
	  ,cm.CLAIM_STATUS

--TODO: REMARKS below-Eliminate NULL values and unnecessary semi-colons
	  ,CONCAT(cm.REMARK_DESCR_1,';',cm.REMARK_DESCR_2,';',cm.REMARK_DESCR_3,';',cm.REMARK_DESCR_4) AS "STATUS_RSN" --TODO-remove # from RSN
--Is the syntax below efficient?
	  ,SUBSTRING(
	  		ISNULL(';'+nullif(ltrim(cm.REMARK_DESCR_1),''), '')+
			ISNULL(';'+nullif(ltrim(cm.REMARK_DESCR_2),''), '')+
			ISNULL(';'+nullif(ltrim(cm.REMARK_DESCR_3),''), '')+
			ISNULL(';'+nullif(ltrim(cm.REMARK_DESCR_4),''), ''), 2, 1000) AS "STATUS_RSN2"  --TODO-remove # from RSN

	  --,cm.HasOtherHealthBenefitPlan AS "COB_FLAG"
	  ,'' AS "NETWORK" -- Currently not supporting this; consider this a placeholder for future enhancement.
      ,cm.PPO_TYPE AS "PAR_STATUS"
	  --,cm.QNXTProviderID AS "BILL_PROV_ID"
	  ,cm.BILLING_PROV_NPI AS "BILL_NPI"
	  ,CASE 
			WHEN cm.BILLING_PROV_GROUP_NAME IS NOT NULL THEN cm.BILLING_PROV_GROUP_NAME
			WHEN cm.BILLING_PROV_GROUP_NAME IS NULL THEN CONCAT(cm.BILLING_PROV_LAST_NAME,', ', cm.BILLING_PROV_FIRST_NAME)
	   END AS "BILL_NM"
	  --,cm.ProviderSpecialty AS "BILL_PROV_SPEC"
	  ,CASE
			WHEN cm.ICD_TYPE = 'ICD10' THEN '10'
			WHEN cm.ICD_TYPE = 'ICD9' THEN '9'
			ELSE ''  -- Currently not reachable (since only values are already called out) but safety valve in case this changes.
	   END AS "ICD_VERSION"	  
	  ,COALESCE(cm.DIAGNOSIS_CODE_1, '') AS "DIAG1"
      ,COALESCE(cm.DIAGNOSIS_CODE_2, '') AS "DIAG2"
      ,COALESCE(cm.DIAGNOSIS_CODE_3, '') AS "DIAG3"
      ,COALESCE(cm.DIAGNOSIS_CODE_4, '') AS "DIAG4"
      ,COALESCE(cm.DIAGNOSIS_CODE_5, '') AS "DIAG5"
      ,COALESCE(cm.DIAGNOSIS_CODE_6, '') AS "DIAG6"
      ,COALESCE(cm.DIAGNOSIS_CODE_7, '') AS "DIAG7"
      ,COALESCE(cm.DIAGNOSIS_CODE_8, '') AS "DIAG8"
      ,COALESCE(cm.DIAGNOSIS_CODE_9, '') AS "DIAG9"
      ,COALESCE(cm.DIAGNOSIS_CODE_10, '') AS "DIAG10"
      ,COALESCE(cm.DIAGNOSIS_CODE_11, '') AS "DIAG11"
      ,COALESCE(cm.DIAGNOSIS_CODE_12, '') AS "DIAG12"
      ,COALESCE(cm.DIAGNOSIS_CODE_13, '') AS "DIAG13"
      ,COALESCE(cm.DIAGNOSIS_CODE_14, '') AS "DIAG14"
      ,COALESCE(cm.DIAGNOSIS_CODE_15, '') AS "DIAG15"
      ,COALESCE(cm.DIAGNOSIS_CODE_16, '') AS "DIAG16"
      ,COALESCE(cm.DIAGNOSIS_CODE_17, '') AS "DIAG17"
      ,COALESCE(cm.DIAGNOSIS_CODE_18, '') AS "DIAG18"
      ,COALESCE(cm.DIAGNOSIS_CODE_19, '') AS "DIAG19"
      ,COALESCE(cm.DIAGNOSIS_CODE_20, '') AS "DIAG20"
      ,COALESCE(cm.DIAGNOSIS_CODE_21, '') AS "DIAG21"
      ,COALESCE(cm.DIAGNOSIS_CODE_22, '') AS "DIAG22"
      ,COALESCE(cm.DIAGNOSIS_CODE_23, '') AS "DIAG23"
      ,COALESCE(cm.DIAGNOSIS_CODE_24, '') AS "DIAG24"
      ,COALESCE(cm.DIAGNOSIS_CODE_25, '') AS "DIAG25"
	  ,'' as "TOS" -- Currently not supporting this; consider this a placeholder for future enhancement.
	  ,CASE
			WHEN cm.BILLED_AMT IS NULL THEN ''
			WHEN cm.BILLED_AMT > 0 THEN CONVERT(VARCHAR(12), cm.BILLED_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "CHARGES"
	  ,CASE
			WHEN cm.PROCESSED_AMT IS NULL THEN ''
			WHEN cm.PROCESSED_AMT > 0 THEN CONVERT(VARCHAR(12), cm.PROCESSED_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "PROCESSED"
	  ,CASE
			WHEN cm.ALLOWED_AMT IS NULL THEN ''
			WHEN cm.ALLOWED_AMT > 0 THEN CONVERT(VARCHAR(12), cm.ALLOWED_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "ALLOWED"
	  ,CASE
			WHEN cm.PAID_AMT IS NULL THEN ''
			WHEN cm.PAID_AMT > 0 THEN CONVERT(VARCHAR(12), cm.PAID_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "PAID"
	  ,CASE
			WHEN cm.COINSURANCE_AMT IS NULL THEN ''
			WHEN cm.COINSURANCE_AMT > 0 THEN CONVERT(VARCHAR(12), cm.COINSURANCE_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "CO_INSUR"
	  ,CASE
			WHEN cm.COPAY_AMT IS NULL THEN ''
			WHEN cm.COPAY_AMT > 0 THEN CONVERT(VARCHAR(12), cm.COPAY_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "COPAY"
	  ,CASE
			WHEN cm.DEDUCTIBLE_AMT IS NULL THEN ''
			WHEN cm.DEDUCTIBLE_AMT > 0 THEN CONVERT(VARCHAR(12), cm.DEDUCTIBLE_AMT)
			ELSE ''  -- If, they want to see NULL, not a number.
	   END AS "DEDUCT"
      ,COALESCE(cm.UNITS_BILLED, '') AS "QUANTITY"
      ,COALESCE(cm.UNITS_PROCESSED, '') AS "QUANTITY_PROCESSED"
	  ,'' AS "HOSPICE_FLAG"  -- Currently not supporting this; consider this a placeholder for future enhancement.
	  ,COALESCE(cm.REVERSAL_CLAIM_KEY, '') AS "REV_CLAIM_KEY"
	  ,COALESCE(cm.ORIGINAL_BRIGHT_CLAIM_ID, '') AS "ORIG_CLAIM_KEY"
	  ,'' AS "MEDICARE_INDICATOR"  -- Currently not supporting this; consider this a placeholder for future enhancement.

  INTO	#for_output
  FROM	#medical_claims AS cm
		INNER JOIN #members AS m
			ON	(cm.BRIGHT_MEMBER_ID = m.BRIGHT_MEMBER_ID);



SELECT  *
FROM #for_output
