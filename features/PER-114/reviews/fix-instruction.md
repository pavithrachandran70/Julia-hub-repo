Fix these forge PR review comments (auto-route):

### Julia comment 4470004872
Required change: in the new sincospi(missing) test, delete the extra checks for sincos, sincosd, sinpi, cospi, and tanpi. This change did not touch those functions, so those tests are duplicates of tests that already exist. Keep only the new checks that sincospi(missing) returns (missing, missing) and that the two values can be split apart.
