# Example:

In `Foundation.hs`:

```haskell
import Yesod.Auth.LDAP

-- stuff

instance YesodAuthLdap App where
    getLdapConfig = do
        extra <- getExtra
        return LDAPConfig
            { usernameFilter = \n -> "mail=" <> n
            , identifierModifier = \n _ -> n
            , ldapUri = unpack $ appLdapUri extra
            , baseDN = Just $ unpack $ appLdapDN extra
            , initDN = unpack $ appLdapAuthDN extra
            , initPass = unpack $ appLdapAuthPass extra
            , ldapScope = LdapScopeSubtree
            }
```

```haskell
instance YesodAuth App where
     -- lots of stuff between here
    authPlugins _ = [authLDAP]
    authenticate (Creds "LDAP" ident extra) = runDB $ do
        x <- getBy $ UniqueUser $ strip ident
        -- some bits settings some app-specific variables used below
        uid <- case x of
            Just (Entity uid _) ->
                return uid
            Nothing -> do
                insert $ (userDefault ident)
                    { userName = fullname
                    , userVerified = True
                    , userIsAdmin = uAdmin
                    , userIsLocked = not uEnable
                    }

        return $ Authenticated uid
```
