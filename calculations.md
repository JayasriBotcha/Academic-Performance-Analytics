## Failure Rate Column = 
DIVIDE(
    'Export'[D's] + 'Export'[F] + 'Export'[W's],
    'Export'[A's] + 'Export'[B's] + 'Export'[C's] +
    'Export'[D's] + 'Export'[F] + 'Export'[W's]
)

## Is Successful (Dynamic) = 
IF(
    SELECTEDVALUE('Export'[% SuccessABC]) >= SELECTEDVALUE('Success Threshold'[Success Threshold]),
    "Yes",
    "No"
)

## Success Threshold Value = SELECTEDVALUE('Success Threshold'[Success Threshold], 0.65)
## EnrollmentExact = 
VAR v =
    CALCULATE(
        SELECTEDVALUE( 'Export'[Course Enrollment] )
    )
RETURN v

## Semester Key = 
VAR Yr =
    VALUE ( RIGHT ( TA[Term Level], 4 ) )
VAR SeasonIdx =
    SWITCH (
        TRUE(),
        CONTAINSSTRING ( TA[Term Level], "Spring" ), 10,
        CONTAINSSTRING ( TA[Term Level], "Fall" ),   80,
        BLANK()
    )
RETURN Yr * 100 + SeasonIdx

## List of Course Listing values = 
VAR __DISTINCT_VALUES_COUNT = DISTINCTCOUNT('TA_ALL'[Course Listing])
VAR __MAX_VALUES_TO_SHOW = 3
RETURN
	IF(
		__DISTINCT_VALUES_COUNT > __MAX_VALUES_TO_SHOW,
		CONCATENATE(
			CONCATENATEX(
				TOPN(
					__MAX_VALUES_TO_SHOW,
					VALUES('TA_ALL'[Course Listing]),
					'TA_ALL'[Course Listing],
					ASC
				),
				'TA_ALL'[Course Listing],
				", ",
				'TA_ALL'[Course Listing],
				ASC
			),
			", etc."
		),
		CONCATENATEX(
			VALUES('TA_ALL'[Course Listing]),
			'TA_ALL'[Course Listing],
			", ",
			'TA_ALL'[Course Listing],
			ASC
		)
	)

## Course_TA_Table = 
VAR L =
    SELECTCOLUMNS (
        'Export',
        "Course Listing", 'Export'[Course Listing],
        "Key", UPPER ( TRIM ( 'Export'[Course Listing] ) )
    )
VAR R =
    SELECTCOLUMNS (
        'TA',
        "TA Name", 'TA'[TA Names],
        "Key", UPPER ( TRIM ( 'TA'[Course Listing] ) )
    ) VAR X = SELECTCOLUMNS( 'Export', "Academic Year", 'Export'[Academic Year], "key", UPPER(TRIM('Export'[Academic Year])))
-- First join Export ↔ TA
VAR J1 = NATURALLEFTOUTERJOIN ( DISTINCT ( L ), DISTINCT ( R ) )
VAR J2 = NATURALLEFTOUTERJOIN ( J1, DISTINCT ( X ) )RETURN
    SELECTCOLUMNS (
        J2,
        "Course Listing", [Course Listing],
        "Academic Year",  [Academic Year],
        "TA Name",        [TA Name]
    )
 
    
