## Distinct subsequences

Key point: 
> Maintain a two-dimensional array which says the answer of the partial input s[0..i] and t[0..j]
> mem[i+1][j+1] = mem[i][j] + mem[i+1][j] if s[i] = t[j] 
> otherwise mem[i+1][j+1] = mem[i+1][j]

##
