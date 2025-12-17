#FSM_counter

## 程式碼
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity FSM_counter is
    generic(
        upper : integer := 9;
        lower : integer := 0
    );
    port(
        rst : in  STD_LOGIC;
        clk : in  STD_LOGIC;
        q   : out STD_LOGIC_VECTOR(3 downto 0)
    );
end FSM_counter;

architecture Behavioral of FSM_counter is
    type counter_state is (counter1, counter2);
    signal now_state, next_state : counter_state;

    signal Qn, Qn_d : integer := lower;
begin

    process(clk, rst)
    begin
        if rst = '1' then
            now_state <= counter1;
        elsif rising_edge(clk) then
            now_state <= next_state;
        end if;
    end process;

    process(now_state, Qn)
    begin
        next_state <= now_state;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    next_state <= counter2;
                end if;

            when counter2 =>
                if Qn <= lower then
                    next_state <= counter1;
                end if;
        end case;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            Qn <= lower;
        elsif rising_edge(clk) then
            Qn <= Qn_d;
        end if;
    end process;

    process(now_state, Qn)
    begin
        Qn_d <= Qn;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    Qn_d <= Qn - 1;
                else
                    Qn_d <= Qn + 1;
                end if;

            when counter2 =>
                if Qn <= lower then
                    Qn_d <= Qn + 1;
                else
                    Qn_d <= Qn - 1;
                end if;
        end case;
    end process;

    q <= std_logic_vector(to_unsigned(Qn, 4));

end Behavioral;

```
## 波形圖
<img width="922" height="578" alt="image" src="https://github.com/user-attachments/assets/27c58ced-f860-4d5d-9803-c4aec2f2cd46" />

## breakdown
<img width="1196" height="528" alt="image" src="https://github.com/user-attachments/assets/a423cfae-7fb3-456c-83a2-56a5ed3a1a85" />


## AOV
<img width="1028" height="352" alt="image" src="https://github.com/user-attachments/assets/37420745-f104-409d-b6fb-fcd9f4fbc5c5" />

## 架構圖
<img width="1296" height="654" alt="image" src="https://github.com/user-attachments/assets/f2857ebd-f5dd-480d-ba81-3d266ff5fdc2" />


## MSC
<img width="1210" height="349" alt="image" src="https://github.com/user-attachments/assets/5dea049a-4c9c-4ece-a6bf-733db939aec6" />

